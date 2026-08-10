Sau khi đối chiếu code SNAP, action Radar > Radiometric > Calibrate chỉ thực hiện radiometric calibration. Nó không tự chạy thermal
  noise removal, GRD border noise removal, terrain flattening hay terrain correction.

## Luồng của action Calibrate

  Người dùng mở Calibrate
    ↓
  SNAP tạo dialog cho operator "Calibration"
    ↓
  Hiển thị tham số theo mission/product
    ↓
  Kiểm tra sản phẩm đầu vào
    ↓
  Chọn calibrator theo mission
    ↓
  Đọc calibration LUT từ metadata
    ↓
  Tạo sản phẩm và các band đích
    ↓
  Nội suy LUT cho từng pixel
    ↓
  Chuyển DN hoặc I/Q thành Sigma0/Beta0/Gamma0
    ↓
  Cập nhật metadata calibration

  ## 1. Mở operator Calibration

  Menu action được đăng ký với:

  displayName = Calibrate
  operatorName = Calibration
  targetProductNameSuffix = _Cal

  Xem /home/hoangnv307/code/snap/microwave-toolbox/sar-op-calibration-ui/src/main/resources/eu/esa/sar/calibration/layer.xml:5.

  Khi người dùng bấm Run, SNAP gọi operator có alias Calibration, tức CalibrationOp.

  ## 2. Điều chỉnh giao diện theo sản phẩm Sentinel‑1

  Đối với Sentinel‑1, dialog:

  - Hiển thị danh sách polarization.
  - Không cho chọn source band tùy ý như các mission thông thường.
  - Vô hiệu hóa lựa chọn auxiliary file.
  - Ẩn Save in dB.
  - Ẩn các tùy chọn tạo Gamma0/Beta0 virtual band kiểu cũ.
  - Hiển thị lựa chọn output trực tiếp:
      - Output sigma0 band
      - Output gamma0 band
      - Output beta0 band

  - Cho phép Save as complex output nếu đầu vào là SLC phức.

  Output DN band tồn tại trong parameter map nhưng checkbox đang bị comment khỏi panel UI.

  Logic giao diện nằm tại /home/hoangnv307/code/snap/microwave-toolbox/sar-op-calibration-ui/src/main/java/eu/esa/sar/calibration/gpf/
  ui/CalibrationOpUI.java:185.

  Mặc định đối với Sentinel‑1:

  outputImageInComplex = false
  outputImageScaleInDb = false
  outputSigmaBand = true
  outputGammaBand = false
  outputBetaBand = false

  Vì vậy kết quả mặc định là Sigma0 dạng tuyến tính.

  ## 3. Kiểm tra phiên bản Sentinel‑1 IPF

  UI đọc ProcessingSystemIdentifier.

  Nếu IPF nhỏ hơn 2.34, SNAP hiển thị cảnh báo:

  Calibration LUT có thể không chính xác,
  kết quả calibration có thể không đáng tin cậy.

  Đây là warning, không phải lỗi bắt buộc dừng. Xem /home/hoangnv307/code/snap/microwave-toolbox/sar-op-calibration-ui/src/main/java/eu/
  esa/sar/calibration/gpf/ui/CalibrationOpUI.java:310.

  ## 4. Ghi các lựa chọn từ dialog vào operator

  Khi chạy, UI chuyển các lựa chọn thành parameters:

  - selectedPolarisations
  - outputSigmaBand
  - outputGammaBand
  - outputBetaBand
  - outputImageInComplex
  - outputImageScaleInDb
  - Auxiliary file parameters đối với các mission khác

  Nếu không chọn polarization, code tự lấy toàn bộ polarization trong metadata.

  Nếu không chọn bất kỳ loại output nào, BaseCalibrator tự bật Sigma0.

  ## 5. Kiểm tra sản phẩm đầu vào

  CalibrationOp kiểm tra:

  - Đây là sản phẩm SAR.
  - Không phải coregistered stack.
  - Chỉ cho complex output nếu đầu vào thực sự là complex.

  Đối với Sentinel1Calibrator, code kiểm tra thêm:

  - Mission là Sentinel‑1.
  - Acquisition mode: IW, EW, SM hoặc WV.
  - Product type: SLC hoặc GRD.
  - Nếu là TOPS SLC phức thì chưa được Deburst.

  Do đó:

  - SM SLC có thể calibration trực tiếp.
  - IW/EW TOPS SLC phải calibration trước Deburst.
  - Không calibration sau khi đã tạo coregistered stack.

  Luồng khởi tạo chung nằm tại /home/hoangnv307/code/snap/microwave-toolbox/sar-op-calibration/src/main/java/eu/esa/sar/calibration/gpf/
  CalibrationOp.java:119.

  ## 6. Chọn calibrator theo mission

  CalibrationOp không chứa một công thức chung cho mọi vệ tinh. Nó gọi CalibrationFactory để chọn implementation theo metadata mission.

  Với Sentinel‑1:

  CalibrationOp
  → CalibrationFactory
  → Sentinel1Calibrator

  Với các mission khác, factory có thể chọn ASAR, ERS, ALOS, Radarsat, TerraSAR-X hoặc calibrator tương ứng.

  ## 7. Tạo sản phẩm đầu ra

  SNAP tạo sản phẩm mới:

  <tên sản phẩm nguồn>_Cal

  Nó:

  - Giữ nguyên kích thước raster.
  - Sao chép metadata, geocoding và các product nodes cần thiết.
  - Tạo band đầu ra theo polarization và loại calibration.

  Ví dụ:

  Sigma0_VV
  Sigma0_VH
  Beta0_VV
  Gamma0_VV

  Với Sentinel‑1, Sigma0, Beta0 và Gamma0 là band được tính trực tiếp từ calibration LUT tương ứng, không phải chỉ là virtual band suy
  ra từ Sigma0.

  ## 8. Đọc calibration LUT từ metadata Sentinel‑1

  SNAP đọc phần:

  Original_Product_Metadata
  └── calibration
      └── calibration data set
          └── calibration
              ├── adsHeader
              └── calibrationVectorList

  Mỗi calibration data set được ghép theo:

  - Polarization.
  - Swath hoặc sub-swath.
  - Start/stop time.

  Mỗi calibration vector chứa:

  - Dòng azimuth.
  - Thời gian.
  - Các vị trí pixel theo range.
  - sigmaNought[].
  - betaNought[].
  - gamma[].
  - dn[].

  Việc đọc vector nằm tại /home/hoangnv307/code/snap/microwave-toolbox/sar-op-calibration/src/main/java/eu/esa/sar/calibration/gpf/
  calibrators/Sentinel1Calibrator.java:187.

  ## 9. Xác định dữ liệu nguồn cho từng band đích

  Đối với GRD, một band đích thường ánh xạ đến một band amplitude hoặc intensity nguồn:

  Amplitude_VV → Sigma0_VV

  Đối với SLC, một band đích intensity ánh xạ đến cặp I/Q:

  i_VV + q_VV → Sigma0_VV

  Nếu chọn complex output, SNAP tạo lại hai band real/imaginary đã được hiệu chỉnh biên độ nhưng vẫn giữ pha.

  ## 10. Chuyển mẫu nguồn thành power

  Trước khi calibration, code đưa dữ liệu về intensity/power.

  Với amplitude:

  $$
  P = DN^2
  $$

  Với SLC complex:

  $$
  P = I^2 + Q^2
  $$

  Với intensity:

  $$
  P = DN
  $$

  Với intensity dB:

  $$
  P = 10^{DN_{\mathrm{dB}}/10}
  $$

  NoData và NaN được giữ lại, không đưa vào tính calibration.

  ## 11. Nội suy calibration LUT

  Calibration LUT không có giá trị cho mọi pixel. Nó chỉ có các điểm mẫu theo range và các vector theo azimuth.

  Với mỗi pixel, SNAP tìm:

  - Hai calibration vector trước và sau theo azimuth.
  - Hai điểm LUT trước và sau theo range.

  Sau đó nội suy song tuyến tính:

  $$
  L(x,y)

  (1-\mu_y)
  \left[
  (1-\mu_x)L_{00}+\mu_xL_{01}
  \right]
  +
  \mu_y
  \left[
  (1-\mu_x)L_{10}+\mu_xL_{11}
  \right]
  $$

  Trong đó:

  - $\mu_x$ là tỷ lệ nội suy theo range.
  - $\mu_y$ là tỷ lệ nội suy theo azimuth.
  - $L$ là LUT tương ứng với Sigma0, Beta0, Gamma0 hoặc DN.

  ## 12. Áp dụng calibration

  Calibration factor được tính:

  $$
  F = \frac{1}{L^2}
  $$

  Giá trị đầu ra:

  $$
  C = P \times F = \frac{P}{L^2}
  $$

  Tùy LUT được chọn:

  $$
  \sigma^0 = \frac{P}{L_{\sigma}^2}
  $$

  $$
  \beta^0 = \frac{P}{L_{\beta}^2}
  $$

  $$
  \gamma^0 = \frac{P}{L_{\gamma}^2}
  $$

  Phần tính theo tile nằm tại /home/hoangnv307/code/snap/microwave-toolbox/sar-op-calibration/src/main/java/eu/esa/sar/calibration/gpf/
  calibrators/Sentinel1Calibrator.java:327.

  ## 13. Trường hợp complex output

  Nếu đầu vào là SLC và bật Save as complex output, SNAP không chỉ xuất power.

  Code tính biên độ calibrated:

  $$
  A_{\mathrm{cal}}

  \sqrt{\frac{I^2+Q^2}{L^2}}
  $$

  Sau đó gắn lại pha của tín hiệu nguồn:

  $$
  I_{\mathrm{cal}}

  A_{\mathrm{cal}}
  \frac{I}{\sqrt{I^2+Q^2}}
  $$

  $$
  Q_{\mathrm{cal}}

  A_{\mathrm{cal}}
  \frac{Q}{\sqrt{I^2+Q^2}}
  $$

  Kết quả:

  - Biên độ được radiometric calibration.
  - Pha được bảo toàn.
  - Vẫn có thể dùng cho xử lý coherent hoặc interferometric sau đó.

  ## 14. Cập nhật metadata

  Sau khi tạo sản phẩm, SNAP đặt:

  abs_calibration_flag = true

  Nếu không xuất complex:

  sample_type = DETECTED

  Các metadata band không thuộc polarization được chọn cũng được loại khỏi target metadata.

  ## Action Calibrate không làm gì?

  Action này không thực hiện:

  - S-1 GRD Border Noise Removal.
  - Thermal Noise Removal.
  - Multilook.
  - Speckle Filter.
  - Terrain Flattening.
  - Terrain Correction.
  - Chuyển hệ tọa độ hoặc geocoding lại ảnh.

  Nó chỉ thực hiện:

  DN hoặc I/Q
  → calibration LUT interpolation
  Với Sentinel‑1 SM SLC, một workflow đầy đủ có thể là:

  Apply Orbit File
  → Thermal Noise Removal
  → Calibrate
  → Multilook nếu cần
  → Terrain Flattening nếu cần
  → Terrain Correction

  Trong đó action Calibrate chính là bước chuyển dữ liệu SLC từ I/Q sang backscatter đã hiệu chỉnh bức xạ.