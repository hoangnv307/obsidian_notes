Đã thêm benchmark đa core vào [band_maths_evaluator_benchmark.cpp](/home/hoangnv307/code/sar_application/benchmark/band_maths_evaluator_benchmark.cpp). Không sửa file `.md`.

Điều kiện đo:

- GCC 13.3, Release `-O3`, WSL2.
- 8 core/16 logical CPU được WSL2 cung cấp.
- 4.194.304 mẫu mỗi workload.
- Các mức 1/2/4/8/16 thread.
- `OMP_PROC_BIND=close`, `OMP_PLACES=cores`.
- Compile expression ngoài vùng đo.
- ExprTk có state riêng cho mỗi thread.
- Mỗi kết quả là trung vị 9 lần sau 2 warm-up.

### Throughput đa core — MPix/s

| Workload    | Engine        | 1 thread | 2 threads | 4 threads | 8 threads | 16 threads |
| ----------- | ------------- | -------: | --------: | --------: | --------: | ---------: |
| Arithmetic  | muParser bulk |   407,93 |    290,00 |    372,15 |    631,88 |     953,41 |
| Arithmetic  | ExprTk OpenMP |   817,41 |  1.568,98 |  2.130,30 |  2.486,72 |   2.525,20 |
| Magnitude   | muParser bulk |   152,56 |    182,86 |    268,09 |    545,83 |     552,43 |
| Magnitude   | ExprTk OpenMP |   448,50 |    891,30 |  1.355,20 |  1.467,33 |   1.277,75 |
| Conditional | muParser bulk |   105,72 |    138,70 |    167,22 |    522,50 |     405,19 |
| Conditional | ExprTk OpenMP |   387,09 |    770,41 |  1.316,68 |  1.467,35 |     918,57 |
| Logarithmic | muParser bulk |    72,60 |    110,20 |    115,35 |    219,79 |     235,58 |
| Logarithmic | ExprTk OpenMP |   149,40 |    295,18 |    558,47 |  1.058,04 |     757,20 |

### Tổng thời gian bốn workload

| Threads | muParser | Scaling | ExprTk | Scaling | ExprTk nhanh hơn |
|---:|---:|---:|---:|---:|---:|
| 1 | 135,216 ms | 1,00× | 53,393 ms | 1,00× | 2,53× |
| 2 | 105,699 ms | 1,28× | 27,032 ms | 1,98× | 3,91× |
| 4 | 88,360 ms | 1,53× | 15,760 ms | 3,39× | 5,61× |
| 8 | 41,432 ms | 3,26× | 11,367 ms | 4,70× | 3,64× |
| 16 | 40,147 ms | 3,37× | 15,049 ms | 3,55× | 2,67× |

Kết luận:

- ExprTk nhanh hơn ở mọi workload và mọi mức thread.
- Cấu hình tốt nhất của ExprTk là 8 thread: `11,367 ms`.
- Cấu hình tốt nhất của muParser là 16 thread: `40,147 ms`.
- So sánh cấu hình tốt nhất của mỗi bên, ExprTk nhanh hơn khoảng `3,53×`.
- ExprTk scale tốt đến 8 thread; lên 16 thread bị giảm với các biểu thức phức tạp do SMT, contention và giới hạn băng thông.
- muParser scale không đều. Với arithmetic, 2 thread còn chậm hơn 1 thread vì phép toán quá nhỏ so với overhead OpenMP.
- Kết quả xác nhận bulk mode dùng OpenMP, nhưng lợi ích từ nhiều core không đủ bù khoảng cách evaluator với ExprTk.

`perf(benchmark): compare multi-core expression throughput`

## Root cause

- Benchmark trước chỉ giới hạn một thread nên chưa đo khả năng scale của muParser bulk và ExprTk khi tận dụng nhiều core.

## Summary

- Thêm OpenMP cho benchmark và workload đa core 4.194.304 mẫu.
- So sánh muParser bulk với ExprTk outer-loop OpenMP tại 1/2/4/8/16 thread.
- Cấp ExprTk expression state riêng cho từng thread để tránh data race.
- Không thay đổi bất kỳ file Markdown nào.

## Test plan

- [x] `cmake -S . -B build-benchmark-fair -G Ninja -DCMAKE_BUILD_TYPE=Release -DSAR_APP_BUILD_BENCHMARKS=ON -DSAR_APP_BUILD_TESTS=OFF` — thành công.
- [x] `cmake --build build-benchmark-fair --target band_maths_evaluator_benchmark -j2` — thành công.
- [x] `OMP_PROC_BIND=close OMP_PLACES=cores ./build-benchmark-fair/benchmark/band_maths_evaluator_benchmark` — chạy hai lần thành công, toàn bộ output hợp lệ.
- [x] `ldd build-benchmark-fair/benchmark/band_maths_evaluator_benchmark` — xác nhận liên kết `libgomp` và `libmuparser`.
- [x] `git diff --check` — thành công.
- [ ] `clang-format` — chưa chạy vì không được cài đặt.