Dựa đúng theo tài liệu **Sentinel-1 Level 1 Detailed Algorithm Definition, Issue 2/5, 17/11/2022** bạn gửi, pipeline Stripmap của Sentinel-1 có thể hiểu ở mức tổng thể là:

**L0 Stripmap → Pre-processing → Doppler Centroid Estimation → SLC Focusing → Internal SLC → L1 Post-processing → L1 SLC hoặc L1 GRD**

Đây cũng chính là kiến trúc IPF ở Figure 3-3. Stripmap là **single-beam, single-swath**, nên pipeline đơn giản hơn IW/EW vì không có các bước riêng của TOPSAR như frequency/time UFR, de-scalloping hay burst merging.

# Các bước xử lý Stripmap L0 → L1

## 1. Pre-processing dữ liệu L0 — Section 4. 
IPF trước hết phân tích raw data, thực hiện internal calibration, kiểm tra header downlink và xây dựng terrain height function. Raw-data analysis sử dụng một phần dữ liệu đã BAQ-decoded để đánh giá thống kê I/Q; với Sentinel-1, do demodulation được thực hiện trong miền số nên I/Q gain imbalance và I/Q non-orthogonality về nguyên tắc không cần như các SAR thế hệ cũ, còn I/Q bias vẫn được xem xét. Internal calibration sử dụng các calibration packet để dựng lại chirp replica, xác định internal delay, tính PG product để bù drift pha/biên độ của thiết bị và xử lý các noise measurements. Header validation kiểm tra các trường như packet counters, BAQ mode, range decimation, PRI, SWST, SWL… đồng thời phát hiện missing lines. Cuối cùng DEM cùng orbit/geometry được dùng để tạo terrain-height function theo azimuth.
    
Một điểm quan trọng là echo raw của Sentinel-1 thường được mã hóa **FDBAQ**; bước decoding chuyển dữ liệu nén này trở lại các mẫu I/Q phức trước khi xử lý tín hiệu. Tài liệu phân biệt Bypass, BAQ 3/4/5-bit và FDBAQ; echo data thông thường dùng FDBAQ.
    
    Có thể hình dung đầu ra của giai đoạn này là:
    
    **L0 ISP packets → signal data + calibration parameters + orbit/attitude + terrain/metadata**
    
2. **Ước lượng Doppler Centroid — Section 5.** Sentinel-1 không chỉ lấy một con số Doppler centroid cố định. Đầu tiên IPF tính **absolute DC từ orbit và attitude** dựa trên hình học satellite–target. Sau đó dữ liệu được decode, raw-data corrected, bù instrument drift và range-compressed để thực hiện **Fine DC Estimation** từ chính echo data. Với Stripmap, bước TOPSAR DCE pre-conditioning được bỏ qua; IPF đi thẳng vào **Correlation Doppler Centroid Estimator (CDCE)**, dùng tương quan giữa các mẫu azimuth liên tiếp.
    
    Fine Doppler thu được bị giới hạn modulo PRF, nên tiếp theo IPF **unwrap Doppler theo range**, sử dụng absolute DC từ geometry để xác định ambiguity number, rồi kết hợp hai nguồn thành **absolute Doppler centroid**. Với single-swath Stripmap, các DC estimate cuối cùng được fit bằng **đa thức bậc hai theo range**; RMS residual của phép fit được dùng làm chỉ tiêu chất lượng DC.
    
    Như vậy output quan trọng của stage này là một hàm dạng:
    
    $f_{DC}(\tau)=c_0+c_1\tau+c_2\tau^2$
    
    cho từng azimuth processing block, chứ không phải chỉ một $f_{DC}$ duy nhất cho toàn ảnh.
    
3. **Range processing để bắt đầu tạo SLC — Section 6.1.** Đây là lúc xử lý image formation thực sự bắt đầu. IPF tạo **Range Reference Function (RRF)** từ chirp replica đã hiệu chuẩn hoặc nominal chirp. RRF về bản chất là matched filter cho tín hiệu chirp phát. Sau đó đối với từng range line, processor thực hiện raw data decoding → raw-data correction → instrument drift compensation. RFI mitigation cũng có thể được bật tùy cấu hình.
    
    Tiếp đó mỗi range line được **range compressed** bằng FFT: zero-pad → range FFT → nhân với RRF → range IFFT → bỏ phần filter transient. Sau compression, IPF áp dụng **range-dependent gain correction**, gồm Elevation Antenna Pattern correction và Range Spreading Loss correction. SWST bias cũng được hiệu chỉnh. Tài liệu còn xử lý riêng trường hợp SWST/SWL thay đổi giữa các pulse bằng cách căn chỉnh các range line và chèn black-fill thích hợp.
    
    Về mặt miền tín hiệu:
    
    $s(\tau,\eta)\xrightarrow{\text{range compression}}s_{RC}(\tau,\eta)$
    
    Lúc này target đã được focus theo **range**, nhưng vẫn chưa được focus theo azimuth.
    
4. **Azimuth pre-processing — Section 6.2.** Dữ liệu range-compressed được chia thành các **azimuth block** có overlap. Với Stripmap, IPF zero-pad theo chiều azimuth rồi thực hiện **Azimuth FFT**:
    
    $s_{RC}(\tau,\eta)\xrightarrow{\mathcal{F}_\eta}S_{RC}(\tau,f_\eta)$
    
    Tức dữ liệu chuyển từ miền **range-time / azimuth-time** sang **range-time / azimuth-frequency**, hay range-Doppler domain. Điểm rất quan trọng: **Stripmap dừng ở đây và đi thẳng sang Range-Doppler Algorithm**; nó không thực hiện Azimuth Frequency UFR vì UFR là TOPSAR-only. Figure 6-1 trong PDF thể hiện nhánh này rất rõ.
    
5. **Secondary Range Compression — SRC, Section 6.3.1.** Range compression ban đầu chưa hoàn toàn chính xác khi Doppler/squint không bằng zero, do range FM thực tế còn phụ thuộc Doppler. Vì vậy Sentinel-1 áp dụng **Secondary Range Compression** trong range-Doppler domain. Processing được thực hiện theo các range segment để có thể coi các tham số range-varying gần như không đổi trong từng segment. Các thao tác chính là range FFT → tạo SRC filter → nhân SRC filter → range IFFT.
    
6. **Range Cell Migration Correction — RCMC, Section 6.3.2.** Trong quá trình vệ tinh bay qua target, slant range tới target thay đổi, nên năng lượng của cùng một target không nằm trong một range bin cố định mà chạy qua nhiều range cell theo azimuth. IPF thực hiện **RCMC** để đưa năng lượng của target trở lại đúng range cell trước khi azimuth focusing.
    
    Đây là một trong ba thành phần cốt lõi của Range-Doppler Algorithm mà tài liệu nêu rõ:
    
    **SRC → RCMC → Azimuth Compression.**
    
7. **Azimuth compression — Section 6.3.4.** Sau RCMC, processor tạo azimuth matched filter dựa trên các thông tin như Doppler centroid $f_{DC}$, azimuth FM rate $K_a$, effective radar velocity và range. Sau khi nhân matched filter trong miền Doppler, IPF thực hiện azimuth IFFT để đưa dữ liệu trở về miền azimuth time.
    
    Có thể biểu diễn khái quát:
    
    $S_{RCMC}(\tau,f_\eta),H_{az}(\tau,f_\eta)\xrightarrow{\mathcal{F}^{-1}_\eta}s_{SLC}(\tau,\eta)$
    
    Tới đây target đã được **focus cả range lẫn azimuth**. Đây chính là **internal SLC image**. Với Stripmap, không có Azimuth Time UFR sau đó vì Section 6.4 là TOPSAR-only. Tài liệu cũng có riêng Section 9.12 và 9.13 để xác định overlap và chiều dài azimuth block cho Stripmap.
    
8. **L1 post-processing — Section 7.** Internal SLC chưa nhất thiết là file L1 cuối cùng. Post-processing quyết định tạo **L1 SLC** hay **L1 GRD**. Figure 7-2 mô tả luồng này gồm range post-processing → azimuth post-processing → output processing.
    
    Ở **range post-processing**, processor FFT theo range, extract spectral look, áp Hamming window rồi IFFT. Nếu output là **SLC thì số range look được đặt bằng 1**. Nếu cần GRD, processor còn xây dựng GR/SR LUT và thực hiện **slant-range → ground-range interpolation** bằng sinc interpolator ở pixel spacing yêu cầu.
    
    Ở **azimuth post-processing**, Stripmap bỏ qua hai bước TOPSAR-only là de-scalloping và de-ramping. Processor thực hiện azimuth FFT → xác định tâm spectrum từ Doppler centroid → extract azimuth look(s) → tính azimuth antenna pattern correction → Hamming weighting → azimuth IFFT. Với **GRD**, sau đó còn azimuth interpolation, detection bằng:
    
    $P=|S|^2=I^2+Q^2$
    
    rồi cộng các azimuth looks và range looks để tạo ảnh multi-look.
    
    Với GRD, thermal-noise vector có thể được nội suy theo azimuth rồi **trừ khỏi power-detected image**. Thermal Noise Removal là GRD-only. Cuối cùng output processing thực hiện square-root extraction đối với GRD khi cấu hình yêu cầu, application scaling và chuyển sang kiểu pixel đầu ra, thông thường 16-bit cho SLC/GRD.
    

Tóm lại, nếu bỏ hết các bước quản lý file/metadata và chỉ nhìn **image formation core**, Stripmap Sentinel-1 thực hiện chuỗi:

**FDBAQ decode → raw correction/calibration → range compression → Doppler centroid estimation → azimuth FFT → SRC → RCMC → azimuth matched filtering/compression → focused SLC**

và nếu muốn **GRD** thì tiếp tục:

**SLC → range/azimuth spectral look extraction + Hamming weighting → multi-looking → slant-to-ground-range conversion → detection → thermal-noise removal → scaling → GRD**.

Điểm đáng chú ý nhất là Sentinel-1 Stripmap về lõi vẫn là một **Range-Doppler Algorithm (RDA)** khá chuẩn, nhưng xung quanh RDA có rất nhiều bước mission-specific: **FDBAQ decoding, internal calibration/PG drift, SWST/SWL handling, EAP correction, range spreading loss, DCE từ geometry + data, azimuth time corrections, radiometric normalization và noise handling**. Đây mới là phần khiến pipeline operational của Sentinel-1 phức tạp hơn một mô hình RDA trong sách giáo khoa.