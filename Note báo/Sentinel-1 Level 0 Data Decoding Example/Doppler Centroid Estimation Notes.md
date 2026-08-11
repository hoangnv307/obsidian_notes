
Nhìn chung, quá trình ước lượng DC sẽ được chia làm các bước như ở Hình 5-1. 
![[Pasted image 20260811093826.png|310]]

### Calculation of Individual Absolute DC Estimates
[[Như nào được gọi là 1 range/azimuth block]]
![[Pasted image 20260811101324.png|534]]
Các bước được đề cập chi tiết ở các mục sau: 
1. Absolute DC calculation (from Orbit & Attitude), được đề cập ở section 5.1
2. Fine DC estimation, được đề cập ở section 5.2
3. Fine DC esimate quality measurement, được đề cập ở section 5.5.1
4. Fine DC esimate unwrapping, được đề cập ở section 5.3
5. Polynominal Fitting, được đề cập ở section 5.5
6. Absolute DC estimation, được đề cập ở section 5.4
Nguồn của tần số DC được sử dụng ở các bước xử lý có thể cấu hình được, và có thể là một trong các giá trị sau: 
- Được tính từ Orbit & Attitude
- Được ước lượng từ data
- Được thiết lập thành một pre-defined, configurable polynomial (trường hợp này không được shown ở Hình 5-2).
Nếu xuất hiện software failure hoặc quality threshold breach, xuất hiện ở bất kì bước nào trong thuật toán trừ bước đầu, the fall-back DC frequency is the one calculated from geometry at the beginning of the procedure.

## 5.1 Absolute DC Calculation (from Orbit & Attitude)

