---
layout: post
title: "How Spark App integrates C++ Eigen functions by Python Bindings?"
date:   2024-02-15 14:30:10 -0500
categories: Engineering
---
Apache Spark is a powerful open-source unified analytics engine known for its speed and ease of use in large-scale data processing. Its flexibility allows integration with various programming languages, including Python, through the PySpark API. However, there are scenarios where you might need the high performance of C++ libraries for numerical computations.
Eigen, a versatile C++ template library for linear algebra, is an excellent choice for such tasks. This blog will guide you through integrating C++ Eigen functions in a Spark application using Python bindings.

# Why Combine Spark, C++, and Eigen?

- Performance: Eigen provides highly optimized operations for matrix and vector computations, which can be crucial for performance-sensitive tasks.
- Scalability: Spark handles large-scale data processing efficiently. By combining it with Eigen, you can perform complex numerical computations on large datasets.
- Ease of Integration: Python bindings (using tools like pybind11) allow seamless integration of C++ code in PySpark applications.

# Prerequisites
Apache Spark: install from source or download latest binary.
Eigen: Download and install Eigen from Eigen's official site.
pybind11: Install pybind11, a lightweight header-only library that exposes C++ types in Python and vice versa. You can install it using pip:
```python
pip install pybind11
```

# Step-by-Step Guide
### 1. Create C++ Function with Eigen
MatrixEigen.h
```cpp
#ifndef MATRIX_EIGEN_H
#define MATRIX_EIGEN_H

#include <Eigen/Dense>
#include <vector>

class MatrixEigen {
public:
    Eigen::VectorXd computeEigenValues(const Eigen::MatrixXd& matrix);
};

#endif // MATRIX_EIGEN_H
```

MatrixEigen.cc
```cpp
#include "MatrixEigen.h"

Eigen::VectorXd MatrixEigen::computeEigenValues(const Eigen::MatrixXd& matrix) {
    Eigen::EigenSolver<Eigen::MatrixXd> solver(matrix);
    Eigen::VectorXd eigenvalues = solver.eigenvalues().real();
    return eigenvalues;
}
```

### 2. Use pybind11 export C++ functions
pybind11_wrapper.cc
```cpp
#include <pybind11/pybind11.h>
#include <pybind11/eigen.h>
#include "MatrixEigen.h"

using namespace pybind11::literals;
namespace py = pybind11;

PYBIND11_MODULE(pybind11_wrapper, m) {
    m.doc() = "Python bindings for MatrixEigen";

    py::class_<MatrixEigen>(m, "MatrixEigen")
        .def(py::init<>())
        .def("compute_ev", &MatrixEigen::computeEigenValues, py::return_value_policy::reference_internal);
}
```

### 3. Compile C++ functions and python wrapper
CMakeLists.txt
```Bash
cmake_minimum_required(VERSION 3.16)
set(CMAKE_CXX_STANDARD 14)

project(MatrixEigen)

# Find Eigen3
find_package(Eigen3  REQUIRED NO_MODULE)
message("EIGEN3_INCLUDE_DIR => ${EIGEN3_INCLUDE_DIR}")

# Include directories
include_directories(${EIGEN3_INCLUDE_DIR})
include_directories(${CMAKE_SOURCE_DIR}/include)

set(SOURCES
        src/MatrixEigen.cc)
# Add the library
add_library(MatrixEigen ${SOURCES})

# Pybind11 setup (brew install pybind11)
find_package(Python3 COMPONENTS Interpreter Development)
find_package(pybind11 REQUIRED)
include_directories(${pybind11_INCLUDE_DIR})

pybind11_add_module(pybind11_wrapper src/pybind11_wrapper.cc)
target_link_libraries(pybind11_wrapper PRIVATE MatrixEigen)

# Install include files
install(FILES include/MatrixEigen.h DESTINATION include)

# Install the library
install(TARGETS MatrixEigen DESTINATION lib)
install(TARGETS pybind11_wrapper DESTINATION lib)
```

run commands below to build MatrixEigen lib and wrapper share lib:
```bash
mkdir _build && \
cd _build && \
cmake .. && \
make
```

### 4. Use Compiled Library in PySpark
Now that we have our C++ function wrapped and compiled, we can use it in a PySpark application.
spark_app.py
```python
from pyspark.sql import SparkSession
import numpy as np
from _build.pybind11_wrapper import MatrixEigen

spark = SparkSession.builder.appName("test").getOrCreate()

# Create a numpy matrix
matrix = np.array([[4.0, -2.0, 5.0], [1.0, 1.0, 3.6], [2.0, 8.0, 11.5]], dtype=np.float64)

# Convert to Eigen matrix (implicitly)
me = MatrixEigen()
eigenvalues = me.compute_ev(matrix)
print("------------------------")
print(eigenvalues)
print("------------------------")

# Stop Spark
spark.stop()
```

### 5. Run PySpark
```bash
export PYSPARK_PYTHON=/usr/local/bin/python
spark/bin/spark-submit spark_app.py
```

Finally, you will get eigenvalues printed in spark app:
```log
......
24/02/25 16:54:24 INFO BlockManager: Using org.apache.spark.storage.RandomBlockReplicationPolicy for block replication policy
24/02/25 16:54:24 INFO BlockManagerMaster: Registering BlockManager BlockManagerId(driver, 10.93.161.214, 62488, None)
24/02/25 16:54:24 INFO BlockManagerMasterEndpoint: Registering block manager 10.93.161.214:62488 with 434.4 MiB RAM, BlockManagerId(driver, 10.93.161.214, 62488, None)
24/02/25 16:54:24 INFO BlockManagerMaster: Registered BlockManager BlockManagerId(driver, 10.93.161.214, 62488, None)
24/02/25 16:54:24 INFO BlockManager: Initialized BlockManager: BlockManagerId(driver, 10.93.161.214, 62488, None)
------------------------
[ 2.62087744 14.67474001 -0.79561745]
------------------------
24/02/25 16:54:24 INFO SparkContext: SparkContext is stopping with exitCode 0.
24/02/25 16:54:24 INFO SparkUI: Stopped Spark web UI at http://10.93.161.214:4040
24/02/25 16:54:24 INFO MapOutputTrackerMasterEndpoint: MapOutputTrackerMasterEndpoint stopped!
24/02/25 16:54:24 INFO MemoryStore: MemoryStore cleared
24/02/25 16:54:24 INFO BlockManager: BlockManager stopped
24/02/25 16:54:24 INFO BlockManagerMaster: BlockManagerMaster stopped
24/02/25 16:54:24 INFO OutputCommitCoordinator$OutputCommitCoordinatorEndpoint: OutputCommitCoordinator stopped!
24/02/25 16:54:24 INFO SparkContext: Successfully stopped SparkContext
24/02/25 16:54:24 INFO ShutdownHookManager: Shutdown hook called
24/02/25 16:54:24 INFO ShutdownHookManager: Deleting directory /private/var/folders/qx/blv38g4d76x03_tkykmfc8kc0000gq/T/spark-bc81631f-9b0c-4cf2-98e6-3a0190c7fb0b/pyspark-c7ebda81-e0e8-47fe-b02c-888d8a5cf441
24/02/25 16:54:24 INFO ShutdownHookManager: Deleting directory /private/var/folders/qx/blv38g4d76x03_tkykmfc8kc0000gq/T/spark-bc81631f-9b0c-4cf2-98e6-3a0190c7fb0b
24/02/25 16:54:24 INFO ShutdownHookManager: Deleting directory /private/var/folders/qx/blv38g4d76x03_tkykmfc8kc0000gq/T/spark-45556a7b-5482-4ea3-addf-65d04a1eba06
```

# Conclusion
By integrating C++ Eigen functions into your Spark applications via Python bindings, you can leverage the power of Eigen's numerical capabilities within the scalable framework of Apache Spark. 
