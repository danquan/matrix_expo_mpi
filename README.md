# Matrix Exponential using MultiProcessing

Tính tích hai ma trận $A, B$; tức là $A\times B$ sử dụng thư viện MPI (C++)

<h1>Yêu cầu:</h1>

- Trình dịch C++ GNU: https://gcc.gnu.org/
- Thư viện \href{https://www.mpich.org/}{mpich}

<h1>Các ý tưởng nhân ma trận đa luồng</h1>

<h2>1. Chia nhỏ bảng ma trận kết quả - Matrix_Expo_1.cpp</h2>

Có $p + 1$ processes thì chia ra làm $p$ phần dọc hoặc ngang tùy ý:

                    00000000000000
                    00000000000000
                    11111111111111
                    11111111111111
                    ...
                    pppppppppppppp
                    pppppppppppppp

Mỗi process sẽ tính một phần như đã chia; tổng độ phức tạp:

- Thời gian: $O(\frac{n^3}{p})$
- Vận chuyển: $O(\frac{mn}{p} + mn)$ [Master -> Worker] + $O(\frac{mn}{p})$ [Worker -> Master]

<h2>2. Giữ nguyên bảng ma trận kết quả, chia nhỏ ma trận A, B - Matrix_Expo_2</h2>

Có $p + 1$ processes thì chia ra A ra làm $p$ phần dọc và B thành $p$ phần ngang:

                    0000000000000
                    0000000000000
                    1111111111111
                    1111111111111
                    ...
                    ppppppppppppp
                    ppppppppppppp

    0011...pp               
    0011...pp
    0011...pp
    0011...pp
    0011...pp
    0011...pp
    0011...pp

Mỗi process sẽ tính (ma trận con của A) * (ma trận con của B) tương ứng mà nó quản lí
Sau đó tính tổng chập của các ma trận mà các process trả về, thu được ma trận đáp án.

Độ phức tạp:

- Thời gian: $O(\frac{n^3}{p})$
- Vận chuyển: $O(\frac{mn}{p} + \frac{mn}{p})$ [Master -> Worker] + $O(mn)$ [Worker -> Master]

Dự đoán cách chia 2 tốt hơn cách chia 1 (Do master phải gửi đi ít thông tin hơn, nên thời gian gửi đi của master ít hơn, còn các worker gửi về song song)

<h2>3. Chia nhỏ ma trận đáp án theo cả dọc và ngang - Matrix_Expo_3</h2>

Với $q = \sqrt{p}$, ma trận sẽ có dạng:

    001122...qq
    001122...qq
    llrr.....hh
    llrr.....hh

- Tức là chia nó thành $q * q$ ma trận nhỏ hơn, sau đó mỗi process quản lí 1 phần và nhân lại
- Ưu điểm: 
    - Tổng độ phức tạp bộ nhớ gửi đi là 2mn/q + mn/p
- Nhược điểm:
    - Không tận dụng được hết các process (Do số process đôi khi không phải số chính phương)
    - Cài đặt async rất khó

<h2>4. Thay đổi mô hình tính toán</h2>

- Thay vì sử dụng mô hình Master-Worker hay Peer-to-Peer, thiết kế ra một mô hình mới
- Mô hình sử dụng chia để trị:

                          Master
                        /        \
                Worker1             Worker2
                /   \              /       \
        Worker3     Worker4     Worker5     Worker6

- Dự đoán sẽ tốt khi có nhiều process

-----------------------------------------------------
<h1>Analysis Matrix_Expo</h1>

<h3>Bộ test với $m, n = 1000$ và $a[i][j] \in [-1, 1]$</h3>



<h3>Bộ test với $m,n = 1000$ và $a[i][j] \in [-1000, 1000]$</h3>



<h3>Bộ test với $m,n = 1000$ và $a[i][j] \in [-10^9, 10^9]$ (testMatrix_test_1000_1000_1e9.INP)</h3>

    - Matrix_Expo_1:
        - Sync: (2 processes): 5722ms, (3 processes): 4650ms, (4 processes): 5600ms, (5 processes): 5416ms, (16 processes): 5188ms
        - Async: (2 processes): 5451ms, (3 processes): 5110ms, (4 processes): 5651ms, (5 processes): 7347ms, (16 processes): 4993ms
    - Matrix_Expo_2:
        - Sync: (2 processes): 4508ms, (3 processes): 3997ms, (4 processes): 4052ms, (5 processes) : 4112ms, (16 processes): 4455ms
        - Async: (2 processes): 4389ms, (3 processes): 3741ms, (4 processes): 4274ms, (5 processes) : 4128ms, (16 processes): 5254ms
    - Matrix_Expo_4 base on idea 1:
        - Sync: (2 processes): 5315ms, (3 processes): 4930ms, (4 processes): 6003ms, (5 processes): 5723ms, (16 processes): 4800ms
        - Async: (2 processes): 5368ms, (3 processes): 4978ms, (4 processes): 6060ms, (5 processes): 5207ms, (16 processes): 4311ms

----------------------------------------------------
