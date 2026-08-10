
Tài liệu tham khảo:
[1] Sentinel-1 Level 0 Data Decoding Example: [Link bài viết](https://github.com/Rich-Hall/sentinel1Level0DecodingDemo/blob/main/sentinel1Level0DecodingDemo.ipynb)
[2] ESA2022_Sentinel-1 Level 1 Detailed Algorithm Definition
[3] Sentinel-1 Space Packet Protocol Data Unit
[4] Các tài liệu của Sentinel-1: [Link](https://sentiwiki.copernicus.eu/web/document-library#DocumentLibrary-SENTINEL-1DocumentsLibrary-S1-Documents)
# 1. Imports và setup 
- Sản phẩm được xét trong bài, chế độ stripmap:
`sao_paulo/s1a-s6-raw-s-vv-20251226t214356-20251226t214427-062491-07d496.dat`

- Lưu lại đường dẫn tới file .dat (file chứa các gói bản tin gửi xuống mặt đất)
# 2. Extract File Metadata
- Một packet thông thường sẽ ba gồm đầu ra của radar instrument tương ứng của một radar echo. 
- Các packets cũng có thể bao gồm các loại dữ liệu khác, ví dụ background noise measurements for instrument calibration. 
- Tác giả sẽ trích metadata từ mỗi packet trên một subset (vì thao tác trên cả ảnh sẽ tốn thời gian).
- Một packet sẽ bao gồm các trường như sau [[Các trường trong một bản tin Sentinel-1]]:
	- Acquistion chunk
	- Packet Number: Index của mỗi packet
	- Packet Version Number
	- Packet Type
	- Secondary Header Flag
	- PID
	- PCAT
	- Sequence Flags
	- Packet Sequence Count
	- Packet Data Length
	- Coarse Time
	- Fine Time
	- Sync
	- Data Take ID
	- ECC Number
	- Test Mode
	- Rx Channel ID
	- Instrument Configuration ID
	- Sub-commutated Ancilliary Data Word Index
	- Sub-commutated Ancilliary Data Word
	- Space Packet Count
	- PRI Count
	- Error Flag
	- BAQ Mode
	- BAQ Block Length
	- Range Decimation
	- Rx Gain
	- Tx Ramp Rate
	- Tx Pulse Start Frequency: $f_0$
	- Tx Pulse Length
	- Rank
	- PRI
	- SWST
	- SWL
	- SAS SSB Flag
	- Polarisation
	- Temperature Compensation
	- Elevation Beam Address
	- Azimuth Beam Address
	- SAS Test Mode
	- Cal Type
	- Calibration Beam Address
	- Calibration Mode
	- Tx Pulse Number
	- Signal Type
	- Swap Flag
	- Swath Number
	- Number of Quads
Tổng cộng ma trận được trích có 46 cột, 50706 hàng tương ứng với số bản tin.
- Ngoài ra, thông tin về quỹ đạo của vệ tinh còn được đính kèm vào các bản tin với tần số thấp hơn, vì vậy tác giả cần thêm một bước để trích xuất trường thông tin này [[Thông tin về quỹ đạo vệ tinh Sentinel-1 được đóng gói trong bản tin]]:
	- X-axis position ECEF
	- Y-axis position ECEF
	- Z-axis position ECEF
	- X-axis velocity ECEF
	- Y-axis velocity ECEF
	- Z-axis velocity ECEF
	- POD Solution Data Timestamp
	- Q0 Attitude Quaternion
	- Q1 Attitude Quaternion
	- Q2 Attitude Quaternion
	- Q3 Attitude Quaternion
	- Omega-X Angular Rate
	- Omega-Y Angular Rate
	- Omega-Z Angular Rate
	- Attitude Data Timestamp
Tổng cộng 15 trường với 762 hàng (số quỹ đạo được đính kèm bản tin) tương ứng.

# 3. Extract data
## 3.1 Select Packets to Process
- Tác giả ở đây chỉ muốn trích xuất những bản tin mang thông tin về radar echoes (dựa vào trường dữ liệu `Signal Type.
- Nhắc lại về khái niệm `Acquistion chunk`, một acquistition chunk là một single continuous segment của SAR acquistition (echoes, noise, calibration, etc.),  where the instrument configuration is fixed and the radar records one uninterrupted sequence.  ==It is one stable, coherent block of data for a given signal type that can be processed as a unit.==
- Tác giả d
```python
# iter_chunks_matching can be passed any constant param you see in the below lists.
# Here, we're using it to iterate over all the chunks which have an echo signal type (as opposed to a noise measurement or a calibration operation)
for chunk in l0file.iter_chunks_matching(signal_type=sentinel1decoder.enums.SignalType.ECHO):
    print(f"Chunk {chunk}:")
    for key, val in l0file.get_acquisition_chunk_constants(chunk).items():
        print(f"\t{key}: {val}")
```

[[num_quads là gì]]
```txt
Chunk 13:
	signal_type: Echo
	swath_num: 6
	num_quads: 9975
	baq_mode: FDBAQ MODE 0
	swst: 8.333617017329499e-05
	swl: 0.00042669824216607814
	pri: 0.000601150045968743
	elevation_beam_address: 5
Chunk 14:
	signal_type: Echo
	swath_num: 6
	num_quads: 9993
	baq_mode: FDBAQ MODE 0
	swst: 8.160444029437422e-05
	swl: 0.00042744421811392094
	pri: 0.000601150045968743
	elevation_beam_address: 5
```

- Trong ví dụ này, tác giả sẽ tập trung vào vùng bờ biển quanh cảng Santos.
```python
selected_chunk = 13
selection = l0file.get_acquisition_chunk_metadata(selected_chunk)
selection
```
Kết quả in ra packet gồm 46 trường dữ liệu như ở trên, với bản tin từ 408 đến 30341, tổng cộng 29934 bản tin echoes.

## 3.2 Extract Raw I/Q Sensor Data
- Tác giả tiến hành trích dữ liệu IQ thành một ma trận dữ liệu 2 chiều, với chiều ngang là fast time $\tau$ và chiều dọc là slow time $\eta$ .
```python
# Decode the IQ data
radar_data = l0file.get_acquisition_chunk_data(selected_chunk)
assert radar_data.dtype == np.complex64

# Cache this data so we can retreive it more quickly next time we want it
l0file.save_acquisition_chunk_data(selected_chunk)
```
- Sau khi vẽ amplitude của dữ liệu IQ thô, ta thấy dữ liệu cũng đã tạo nên một cấu trúc địa hình cụ thể, nhưng ta chưa thể phân biệt/nhận dạng được các vật thể. 
```python
# Plot the raw IQ data extracted from the data file
plt.figure(figsize=(12, 12))
plt.title("Sentinel-1 Raw I/Q Sensor Output")
# We're just going to plot every 20th row/col value for speed here
plt.imshow(abs(radar_data[::20,::20]), vmin=0, vmax=15, origin='lower')
plt.xlabel("Fast Time (down range)")
plt.ylabel("Slow Time (cross range)")
plt.show()
```
![[Pasted image 20260805181828.png]]

# 4. Image Processing
- Ở phần này, tác giả sẽ thực thi các bước của thuật toán Range-Doppler:
	1. Range Compression
	2. Transform to Range-Doppler domain 
	3. Range Cell Migration Correction (RCMC)
	4. Azimuth compression
	5. Transform to time domain
	6. Image formation
## 4.1 Define auxiliary parameters
- Tác giả định nghĩa các tham số cần cho quá trình lấy nét ảnh như sau: 
	- Kích thước ảnh 
	- Các tham số của xung truyền sử dụng để tổng hợp một xung replica Tx
	- Sample rates theo chiều range và azimuth 
	- Giá trị fastime $\tau$ tương ứng với mỗi range sample dọc theo một range line, và slant range tương ứng của closest approach $R_0$ for each of these range samples.
	- Giá trị tần số theo trục range $f_\tau$ và aizmuth $f_\eta$  sau khi dữ liệu được chuyển qua miền tần số
	- Vận tốc vệ tinh hiệu dụng ($V_r$), với $V_r \approx \sqrt{V_s Vg}$, với $V_s$ giá trị vector vận tốc vệ tinh định mức, $V_g$ là vận tốc búp sóng ăng ten trên mặt đất. Công thức này dựa trên báo [link](https://iopscience.iop.org/article/10.1088/1757-899X/1172/1/012012/pdf), lưu ý là $V_g$ và vì thế là $V_r$ sẽ thay đổi theo slant range. 
Ý nghĩa một vài trường: 
- Range Decimation (RGDEC): [[Các trường trong một bản tin Sentinel-1#Range Decimation]]
- Rank: [[Các trường trong một bản tin Sentinel-1#Rank]]
```python
# Image sizes
len_range_line = radar_data.shape[1]
len_az_line = radar_data.shape[0]

# Tx pulse parameters
c = sentinel1decoder.constants.SPEED_OF_LIGHT_MPS
RGDEC = selection["Range Decimation"].unique()[0]
PRI = selection["PRI"].unique()[0]
rank = selection["Rank"].unique()[0]
suppressed_data_time = 320/(8*sentinel1decoder.constants.F_REF)
range_start_time = selection["SWST"].unique()[0] + suppressed_data_time
wavelength_m = sentinel1decoder.constants.TX_WAVELENGTH_M

# Sample rates
range_sample_freq = sentinel1decoder.utilities.range_dec_to_sample_rate(RGDEC)
range_sample_period = 1/range_sample_freq
az_sample_freq = 1 / PRI
az_sample_period = PRI

# Fast time vector - defines the time axis along the fast time direction
sample_num_along_range_line = np.arange(0, len_range_line, 1)
fast_time_vec = range_start_time + (range_sample_period * sample_num_along_range_line)

# Slant range vector - defines R0, the range of closest approach, for each range cell
slant_range_vec_m = ((rank * PRI) + fast_time_vec) * c/2
    
# Axes - defines the frequency axes in each direction after FFT
az_freq_vals_hz = np.arange(-az_sample_freq/2, az_sample_freq/2, 1/(PRI*len_az_line))
 
# Spacecraft velocity - numerical calculation of the effective spacecraft velocity
ecef_vels = l0file.ephemeris.apply(lambda x: math.sqrt(x["X-axis velocity ECEF"]**2 + x["Y-axis velocity ECEF"]**2 +x["Z-axis velocity ECEF"]**2), axis=1)
velocity_interp = interp1d(l0file.ephemeris["POD Solution Data Timestamp"].unique(), ecef_vels.unique(), fill_value="extrapolate")
space_velocities_mps = selection.apply(lambda x: velocity_interp(x["Coarse Time"] + x["Fine Time"]), axis=1).to_numpy().astype(float)

# Calculate required FFT size for linear convolution in range compression
# This ensures N >= L + M - 1, where L is signal length and M is filter length
TXPL = selection["Tx Pulse Length"].unique()[0]
num_tx_vals = int(TXPL*range_sample_freq)
required_fft_size = len_range_line + num_tx_vals - 1
# Round up to next power of 2 for FFT efficiency
range_fft_size = int(2**np.ceil(np.log2(required_fft_size)))

# Delete arrays we no longer need
del velocity_interp
del ecef_vels
```
- Sau đó tác giả tính cosine của góc squint tức thời $D(f\eta, V_r)$, với
$$
D(f_\eta, V_r) = \sqrt{ 1 - \frac{c^2 f_\eta ^2}{4V_r^2f_0^2} }
$$
giá trị này phụ thuộc vào cả azimuth và range, và vì thế là một mảng 2 chiều có kích thước bằng với data radar của chúng ta. Nếu lưu thêm 1 array đó thì sẽ rất tốn bộ nhớ. Vì thế, tác giả tạo một vòng lặp trả về các chunks nhỏ hơn của array. 
```python
# The ephemeris is provided at a much lower rate than the rate we receive packets
# Therefore, we're using interpolation to get the spacecraft position along the azimuth axis
x_interp = interp1d(l0file.ephemeris["POD Solution Data Timestamp"].unique(), l0file.ephemeris["X-axis position ECEF"].unique(), fill_value="extrapolate")
y_interp = interp1d(l0file.ephemeris["POD Solution Data Timestamp"].unique(), l0file.ephemeris["Y-axis position ECEF"].unique(), fill_value="extrapolate")
z_interp = interp1d(l0file.ephemeris["POD Solution Data Timestamp"].unique(), l0file.ephemeris["Z-axis position ECEF"].unique(), fill_value="extrapolate")
x_positions = selection.apply(lambda x: x_interp(x["Coarse Time"] + x["Fine Time"]), axis=1).to_numpy().astype(float)
y_positions = selection.apply(lambda x: y_interp(x["Coarse Time"] + x["Fine Time"]), axis=1).to_numpy().astype(float)
z_positions = selection.apply(lambda x: z_interp(x["Coarse Time"] + x["Fine Time"]), axis=1).to_numpy().astype(float)
position_array = np.transpose(np.vstack((x_positions, y_positions, z_positions)))

wgs84_semi_major_axis_m = sentinel1decoder.constants.WGS84_SEMI_MAJOR_AXIS_M
wgs84_semi_minor_axis_m = sentinel1decoder.constants.WGS84_SEMI_MINOR_AXIS_M
satellite_distance_from_center_m = np.linalg.norm(position_array, axis=1)
satellite_angular_velocity_rps = np.divide(space_velocities_mps, satellite_distance_from_center_m)
satellite_latitude_rad = np.arctan(np.divide(position_array[:, 2], position_array[:, 0]))
local_earth_rad_m = np.sqrt(
    np.divide(
        (np.square(wgs84_semi_major_axis_m**2 * np.cos(satellite_latitude_rad)) + np.square(wgs84_semi_minor_axis_m**2 * np.sin(satellite_latitude_rad))),
        (np.square(wgs84_semi_major_axis_m * np.cos(satellite_latitude_rad)) + np.square(wgs84_semi_minor_axis_m * np.sin(satellite_latitude_rad)))
    )
)

# Delete variables we no longer need
del x_interp
del y_interp
del z_interp
del satellite_angular_velocity_rps
del satellite_latitude_rad
del position_array
del x_positions
del y_positions
del z_positions
gc.collect()

def compute_D_chunks(local_earth_rad_m, satellite_distance_from_center_m, space_velocities_mps, slant_range_vec_m, 
                     az_freq_vals_hz, wavelength_m, chunk_size=512):
    """
    Generator that yields chunks of D on demand.
    Each chunk computes intermediate arrays only for that chunk's rows.
    This avoids storing the full 2D D array in memory.
    
    Yields:
        (start_idx, end_idx, D_chunk) tuples where:
        - start_idx, end_idx: slice indices for this chunk
        - D_chunk: 2D array of D values for this chunk, shape (chunk_size, len_range_line)
    """
    len_az_line = len(local_earth_rad_m)
    
    for start_idx in range(0, len_az_line, chunk_size):
        end_idx = min(start_idx + chunk_size, len_az_line)
        
        # Extract chunk of 1D arrays
        local_earth_rad_chunk_m = local_earth_rad_m[start_idx:end_idx]
        satellite_distance_chunk_m = satellite_distance_from_center_m[start_idx:end_idx]
        space_velocities_chunk_mps = space_velocities_mps[start_idx:end_idx]
        az_freq_vals_chunk_hz = az_freq_vals_hz[start_idx:end_idx]
        
        # Compute angular velocity for this chunk (angular_velocity = space_velocities / distance_from_center)
        satellite_angular_velocity_chunk_rps = np.divide(space_velocities_chunk_mps, satellite_distance_chunk_m)
        
        # Compute 2D intermediate arrays for this chunk only
        cos_beta_chunk = (np.divide(
            np.square(local_earth_rad_chunk_m[:, np.newaxis]) + 
            np.square(satellite_distance_chunk_m[:, np.newaxis]) - 
            np.square(slant_range_vec_m), 
            2 * local_earth_rad_chunk_m[:, np.newaxis] * satellite_distance_chunk_m[:, np.newaxis]
        ))
        ground_velocities_chunk_mps = (local_earth_rad_chunk_m[:, np.newaxis] * 
                                   satellite_angular_velocity_chunk_rps[:, np.newaxis] * cos_beta_chunk)
        effective_velocities_chunk_mps = np.sqrt(space_velocities_chunk_mps[:, np.newaxis] * 
                                            ground_velocities_chunk_mps)
        
        # Compute D chunk
        D_chunk = np.sqrt(1 - np.divide(
            wavelength_m**2 * np.square(az_freq_vals_chunk_hz[:, np.newaxis]),
            4 * np.square(effective_velocities_chunk_mps)
        ))
        
        yield start_idx, end_idx, D_chunk
        # Chunk is automatically garbage collected after yield

```

## 4.2 Convert data to 2D frequency domain
- Ở bước này, tác giả FFT theo cả 2 trục azimuth và range.
```python
# Zero-pad along range axis to enable linear convolution
radar_data = np.pad(radar_data, ((0, 0), (0, range_fft_size - len_range_line)), mode='constant', constant_values=0)

# FFT each range line with explicit size for linear convolution
radar_data = fft(radar_data, n=range_fft_size, axis=1, overwrite_x=True)

# FFT each azimuth line
radar_data = fftshift(fft(radar_data, axis=0, overwrite_x=True), axes=0)

assert radar_data.dtype == np.complex64
```

## 4.3 Range compression - create and apply matched filter
- Tác giả dùng bản sao xung Tx được lấy thông số từ gói bản tin metadata. Vì chúng ta đang ở miền tần số, chúng ta cũng phải chuyển bản sao xung sang miền tần số để dùng matched filter, sau đó lấy liên hợp phức. Cuối cùng, nhân từng range line với bộ lọc khớp. 
- Bản sao xung Tx có dạng:
$$
TxPulse = exp\left\{2i\pi((TXPSF + \frac{TXPRR \times TXPL}{2})\tau + \frac{TXPRR}{2}\tau^2)\right\}
$$
với TXPSF là tần số bắt đầu của xung Tx, TXPRRR là ramp rate của xung Tx, TXPL là độ dài xung Tx. 
```python
# Create replica pulse
TXPSF = selection["Tx Pulse Start Frequency"].unique()[0]
TXPRR = selection["Tx Ramp Rate"].unique()[0]
TXPL = selection["Tx Pulse Length"].unique()[0]
num_tx_vals = int(TXPL*range_sample_freq)
tx_replica_time_vals = np.linspace(-TXPL/2, TXPL/2, num=num_tx_vals)
phi1 = TXPSF + TXPRR*TXPL/2
phi2 = TXPRR/2
tx_replica = np.exp(2.0j * np.pi * (phi1*tx_replica_time_vals + phi2*tx_replica_time_vals**2)).astype(np.complex64)

# Create range filter from replica pulse, padded to range_fft_size for linear convolution
range_filter = np.zeros(range_fft_size, dtype=np.complex64)
index_start = np.ceil((range_fft_size-num_tx_vals)/2)-1
index_end = num_tx_vals+np.ceil((range_fft_size-num_tx_vals)/2)-2
range_filter[int(index_start):int(index_end+1)] = tx_replica
range_filter = np.conjugate(fft(range_filter, n=range_fft_size, overwrite_x=True))

# Apply filter
radar_data = np.multiply(radar_data, range_filter)

del range_filter
del tx_replica
gc.collect()

assert radar_data.dtype == np.complex64
```

## 4.4 Range cell migration corection 
- Vì chuyển động của bộ thu có liên quan đến rằng buộc giữa thông tin range và azimuth, các mục tiêu điểm sẽ có xu hướng trải dài theo hình vòng cung qua những range bins cũng như theo chiều azimuth. 
- Vì vậy ta cần áp dụng một phép dịch chuyển để xếp đều phase history liên quan của each pointlike target thành một range bin, để ta có thể thao tác trên 1 chiều dọc theo trục azimuth để thực hiện azimuth compression. 
- The RCMC shift được định nghĩa như sau: 
$$
RCMC\; shift = R_0(\frac{1}{D} -1)
$$
với D là cosine của góc squint tức thời và $R_0$ là the range of closest approach (được đề cập ở mục 4.1). Vì chúng ta đang thực hiện trên miền tần số, ta cần áp dụng một bộ lọc dưới dạng:
$$
RCMC \; filter = exp\left\{4i\pi\frac{f_\tau}{c}(RCMC\; shift)\right\}
$$
- Bộ lọc này cần được nhân với mọi range line trong dữ liệu.
```python
# Create frequency axis for RCMC (corresponding to original range samples only)
range_freq_vals_hz_rcmc = np.linspace(-range_sample_freq/2, range_sample_freq/2, num=len_range_line)

# Process RCMC filter in chunks to avoid storing full D array
for start_idx, end_idx, D_chunk in compute_D_chunks(local_earth_rad_m, satellite_distance_from_center_m, space_velocities_mps, 
                                                     slant_range_vec_m, az_freq_vals_hz, wavelength_m):
    # Compute RCMC filter for this chunk (using frequency axis matching original range samples)
    rcmc_shift_chunk_m = slant_range_vec_m[0] * (np.divide(1, D_chunk) - 1)
    # rcmc_shift_chunk_m shape: (chunk_size, len_range_line)
    # range_freq_vals_hz_rcmc shape: (len_range_line,)
    # We need to broadcast: (len_range_line,) * (chunk_size, len_range_line) -> (chunk_size, len_range_line)
    rcmc_filter_chunk = np.exp(4.0j * np.pi * range_freq_vals_hz_rcmc[np.newaxis, :] * rcmc_shift_chunk_m / c).astype(np.complex64)
    
    # Pad RCMC filter to match padded frequency domain size (fill with 1.0 for padded bins)
    rcmc_filter_chunk_padded = np.ones((rcmc_filter_chunk.shape[0], range_fft_size), dtype=np.complex64)
    rcmc_filter_chunk_padded[:, :len_range_line] = rcmc_filter_chunk
    
    # Apply RCMC filter to this chunk
    radar_data[start_idx:end_idx, :] *= rcmc_filter_chunk_padded

del D_chunk
del rcmc_filter_chunk
del rcmc_filter_chunk_padded
del range_freq_vals_hz_rcmc
gc.collect()

assert radar_data.dtype == np.complex64
```

## 4.5 Convert to Range-Doppler domain
- Ta đã thực hiện xong việc xử lý dữ liệu trên miền range, vậy nên ta có thể inverse FFT lại miền range dọc theo trục range. Dữ liệu vẫn ở miền tần số ở chiều azimuth.
```python
radar_data = ifftshift(ifft(radar_data, axis=1, overwrite_x=True), axes=1)

# Truncate back to original range line length after linear convolution
radar_data = radar_data[:, :len_range_line]

assert radar_data.dtype == np.complex64
```

## 4.6 Azimuth compression - create and apply matched filter.
- Bộ lọc theo chiều aizmuth được định nghĩa dưới dạng: 
$$
Azimuth \; filter = exp\left\{4i\pi\frac{R_0D(f_\eta,V_r)}{\lambda}\right\}
$$
```python
# Process azimuth filter in chunks to avoid storing full D array
for start_idx, end_idx, D_chunk in compute_D_chunks(local_earth_rad_m, satellite_distance_from_center_m, space_velocities_mps, 
                                                     slant_range_vec_m, az_freq_vals_hz, wavelength_m):
    # Compute azimuth filter for this chunk
    az_filter_chunk = np.exp(4.0j * np.pi * slant_range_vec_m * D_chunk / wavelength_m).astype(np.complex64)
    
    # Apply filter to this chunk
    radar_data[start_idx:end_idx, :] *= az_filter_chunk

# Clean up arrays we no longer need
del D_chunk
del az_filter_chunk
del local_earth_rad_m
del satellite_distance_from_center_m
del space_velocities_mps
del slant_range_vec_m
del az_freq_vals_hz
del fast_time_vec
gc.collect()

assert radar_data.dtype == np.complex64
```

## 4.7 Transform back to range-azimuth domain
- Cuối cùng, tác giả chuyển đổi miền frequency bằng cách lấy IFFT each azimuth line. 
```python
radar_data = ifft(radar_data, axis=0, overwrite_x=True) 
assert radar_data.dtype == np.complex64
```

# 5. Plot Results
- Kết quả của chúng ta sau xử lý: 
```python
# Plot final image
plt.figure(figsize=(12, 12))
plt.title("Sentinel-1 Processed SAR Image")
plt.imshow(abs(radar_data[::20, ::20]), origin='lower', norm=colors.LogNorm(vmin=300, vmax=10000))
plt.xlabel("Down Range (samples)")
plt.ylabel("Cross Range (samples)")
plt.show()
```
![[Pasted image 20260806085858.png]]
- Zoom in: 
```python
# Plot final image - detail
plt.figure(figsize=(12, 12))
plt.title("Sentinel-1 Processed SAR Image - detail")
plt.imshow(abs(radar_data[9000:11000, 6000:8000]), origin='lower', norm=colors.LogNorm(vmin=300, vmax=10000))
plt.xlabel("Down Range (samples)")
plt.ylabel("Cross Range (samples)")
plt.show()
```
![[Pasted image 20260806090115.png]]
- Kết quả ảnh vẫn chưa phải là perfectly. Ở bài test này, tác giả đang giả sử rằng Doppler Controid bằng 0Hz, cũng như chưa áp dụng một số bước bổ sung mà ESA đã sử dụng để tạo ảnh L1, ví dụng Secondary Range Compression (SRC). 