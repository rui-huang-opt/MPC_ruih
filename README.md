# MPC_ruih
Arduino library for linear MPC controller.
## References
This algorithm refers to the article [*`An Accelerated Dual Gradient-Projection Algorithm for Embedded Linear Model Predictive Control`*](https://ieeexplore.ieee.org/document/6426458) by Panagiotis Patrinos and Alberto Bemporad. You can read the [article](https://ieeexplore.ieee.org/document/6426458) for more details.
## Usage
Please see [matrixTest](https://github.com/rhrhhrhr/MPC_ruih/blob/main/examples/matrixTest/matrixTest.ino) if you want to know how to use the Matrix class and see [mpcTest](https://github.com/rhrhhrhr/MPC_ruih/blob/main/examples/mpcTest/mpcTest.ino) for MPC class.
### Parameters of the Matrix class
```cpp
uint32_t row, column;             // row and column of matrix 矩阵的行和列
MatDataType_t data[MAXSIZE] = {};      // data of matrix 矩阵的数据
```
The data of the matrix is stored in a fixed length array `data[MAXSIZE]`. You could modify the `MAXSIZE` in [MPCConfig.h](https://github.com/rhrhhrhr/MPC_ruih/blob/main/src/MPCConfig.h).
### Initialize a matrix
You can initialize a matrix in four ways: without parameters, with row and column, by an array or by another matrix.
#### Without parameters
This operation will generate an all zero matrix with one row and one column<br><br>
**code:**
```cpp
Matrix mat;
mat.Print();
```
**result:**
```cpp
0
```
#### With row and column
This operation will generate an all zero matrix with the corresponding row and column<br><br>
**code:**
```cpp
Matrix mat = Matrix(2, 2);
mat.Print();
```
**result:**
```cpp
0 0 
0 0 
```
#### By an array
This operation will push the data of the array into the matrix.<br><br>
**code:**
```cpp
MatDataType_t arr[4] = {1, 2, 3, 4};
Matrix mat = Matrix(2, 2, arr);
mat.Print();
```
**result:**
```cpp
1 2 
3 4 
```
Note that this method can only be used when the row and column of the matrix have been determined, or the assignment won't success. The same data with different rows and columns will represent different matrices.<br><br>
**code:**
```cpp
MatDataType_t arr[4] = {1, 2, 3, 4};
Matrix mat1 = Matrix(2, 2, arr);
Matrix mat2 = Matrix(4, 1, arr);
mat1.Print();
mat2.Print();
```
**result:**
```cpp
1 2 
3 4 

1 
2 
3 
4 
```
#### By another matrix
This operation will copy the row, column and data of another matrix.<br><br>
**code:**
```cpp
MatDataType_t arr[4] = {1, 2, 3, 4};
Matrix mat1 = Matrix(2, 2, arr);
Matrix mat2 = Matrix(4, 1);

mat1.Print();
mat2.Print();

mat2 = mat1;
mat2.Print();
```
**result:**
```cpp
1 2 
3 4 

0 
0 
0 
0 

1 2 
3 4 
```
### Parameters of the MPC class
```cpp
MatDataType_t L_phi, epsilon_V, epsilon_g;
uint32_t max_iter, N;
Matrix A, B, Q, R, QN, F, G, c, FN, cN;
```
Parameter `L_phi` determines the step size of gradient descent. It is best to calculate it through some methods rather than taking arbitrary values. The corresponding code about how to calculate it in python has been uploaded in repo [MPC_ruih_MPCSetup](https://github.com/rhrhhrhr/MPC_ruih_MPCSetup). For more details you can refer to the article [*An Accelerated Dual Gradient-Projection Algorithm for Embedded Linear Model Predictive Control*](https://ieeexplore.ieee.org/document/6426458).<br><br>
Parameters `epsilon_V` and `epsilon_g` here are the tolerances of the error between optimal cost and real cost and the violation of constraints respectively. Note that the tolerances not only describe the absolute error, but also describe the relative error sometimes. This depends on the specific situation.<br><br>
Parameter `max_iter` is the maximum number of the solving step.<br><br>
Parameter `N` is the prediction horizon of the MPC controller.<br><br>
Matrix `A`, `B` describe the system's state space equation $x_{k+1} = Ax_k + Bu_k$<br><br>
Matrix `Q`, `R`, `QN` describe the cost function of the MPC controller $V(X, U) = \sum\limits_{k=0}^{N-1}(x_k^TQx_k + u_k^TRu_k) + x_N^TQ_Nx_N$<br><br>
Matrix `F`, `G`, `c` describe the state and input constraints $Fx_k + Gu_k \le c$<br><br>
Matrix `FN`, `cN` describe the terminal constraints $F_Nx_N \le c_N$
### Initialize a MPC controller
You can use the python package LinearMPCFactor in repo [MPC_ruih_MPCSetup](https://github.com/rhrhhrhr/MPC_ruih_MPCSetup) to generate the setup code for a MPC controller as below:<br><br>
**code:**
```python
import numpy as np
import LinearMPCFactor as lMf

if __name__ == '__main__':
    A = np.array([[2, 1], [0, 2]])  # state space equation A 状态空间方程中的A
    B = np.array([[1, 0], [0, 1]])  # state space equation B 状态空间方程中的B
    Q = np.array([[1, 0], [0, 3]])  # cost function Q, which determines the convergence rate of the state 代价函数中的Q，决定了状态的收敛速度
    R = np.array([[1, 0], [0, 1]])  # cost function R, which determines the convergence rate of the input 代价函数中的R，决定了输入的收敛速度

    A_x = np.array([[1, 0], [-1, 0]])  # state constraints A_x @ x_k <= b_x 状态约束 A_x @ x_k <= b_x
    b_x = np.array([5, 5])

    A_u = np.array([[1, 0], [-1, 0], [0, 1], [0, -1]])  # input constraints A_u @ u_k <= b_u 输入约束 A_u @ u_k <= b_u
    b_u = np.array([1, 1, 1, 1])

    N = 5  # prediction horizon 预测区间

    mpc = lMf.LinearMPCFactor(A, B, Q, R, N, A_x, b_x, A_u, b_u)  # print the cpp code for initializing a MPC class 打印出初始化MPC类的cpp代码

    # mpc.decPlace = 4  # This is the number of decimal places reserved for matrix data, which defaults to 6 decimal places 这是矩阵数据保留的小数位数，默认保留小数点后6位
    e_V = 0.001  # tolerance of the error between optimal cost and real cost 实际代价函数的值与最优之间的最大误差
    e_g = 0.001  # tolerance of the violation of constraints 最大的违反约束的程度
    max_iter = 1000  # maximum steps of the solver 最大迭代步数

    mpc.PrintCppCode(e_V, e_g, max_iter)
```
**result:**
```cpp
MatDataType_t L_phi = 9.90287;
MatDataType_t e_V = 0.001;
MatDataType_t e_g = 0.001;
uint32_t max_iter = 1000;
uint32_t N = 5;

MatDataType_t A_arr[4] = {2, 1, 0, 2};
MatDataType_t B_arr[4] = {1, 0, 0, 1};
MatDataType_t Q_arr[4] = {1, 0, 0, 3};
MatDataType_t R_arr[4] = {1, 0, 0, 1};
MatDataType_t QN_arr[4] = {4.167039, 1.756553, 1.756553, 7.455801};
MatDataType_t F_arr[12] = {1.0, 0.0, -1.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0};
MatDataType_t G_arr[12] = {0.0, 0.0, 0.0, 0.0, 1.0, 0.0, -1.0, 0.0, 0.0, 1.0, 0.0, -1.0};
MatDataType_t c_arr[6] = {5, 5, 1, 1, 1, 1};
MatDataType_t FN_arr[8] = {1.583519, 0.878277, -1.583519, -0.878277, 0.086517, 1.788762, -0.086517, -1.788762};
MatDataType_t cN_arr[4] = {1.0, 1.0, 1.0, 1.0};

Matrix A = Matrix(2, 2, A_arr);
Matrix B = Matrix(2, 2, B_arr);
Matrix Q = Matrix(2, 2, Q_arr);
Matrix R = Matrix(2, 2, R_arr);
Matrix QN = Matrix(2, 2, QN_arr);
Matrix F = Matrix(6, 2, F_arr);
Matrix G = Matrix(6, 2, G_arr);
Matrix c = Matrix(6, 1, c_arr);
Matrix FN = Matrix(4, 2, FN_arr);
Matrix cN = Matrix(4, 1, cN_arr);

MPC mpc = MPC(L_phi, e_V, e_g, max_iter, N, A, B, Q, R, QN, F, G, c, FN, cN);
```
## Note
### Matrix size
Considering the real-time requirements of the algorithm, the data of the matrix is stored in a fixed length array rather than vector container. If larger matrix operations are needed, you can modify the `MAXSIZE` in [MPCConfig.h](https://github.com/rhrhhrhr/MPC_ruih/blob/main/src/MPCConfig.h), which is the maximum number of matrix elements. 
### Matrix data type
You can also determine whether the data type of the matrix is float or double through the `SINGLE_PRECISION` and `DOUBLE_PRECISION` in [MPCConfig.h](https://github.com/rhrhhrhr/MPC_ruih/blob/main/src/MPCConfig.h).
### Order of matrix multiplication
Generally, the implementation of matrix multiplication is as follows:
```cpp
for (int i = 0; i < this->row; i++) {
    for (int j = 0; j < mat.column; j++) {
        for (int k = 0; k < this->column; k++) {
            temp.data[i * mat.column + j] +=
                    this->data[i * this->column + k] * mat.data[k * mat.column + j];
        }
    }
}
```
However, the speed of matrix multiplication can be improved by changing the order of i, j, k because normal order can cause discontinuous array memory access. But unexpectedly, testing on ESP32 shows that the order of jki is the fastest. The specific reason is not yet clear, so you can try which one is faster when using it yourself by using the related macro definition in [MPCConfig.h](https://github.com/rhrhhrhr/MPC_ruih/blob/main/src/MPCConfig.h).
```cpp
//#define ORDER_OF_MATMUL_IJK
//#define ORDER_OF_MATMUL_IKJ
//#define ORDER_OF_MATMUL_JIK
#define ORDER_OF_MATMUL_JKI
//#define ORDER_OF_MATMUL_KIJ
//#define ORDER_OF_MATMUL_KJI
```
## Licence
MIT
## Author
Rui Huang
