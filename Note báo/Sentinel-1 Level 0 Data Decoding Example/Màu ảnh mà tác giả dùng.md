Nếu bạn đang hỏi **dải màu trong notebook `sentinel1Level0DecodingDemo.ipynb`** mà bạn đã gửi link , cụ thể đoạn:

```python
plt.imshow(abs(radar_data[::20,::20]), vmin=0, vmax=15, origin='lower')
```

thì tác giả **không truyền tham số `cmap`**.

Với `matplotlib.pyplot.imshow()`, khi không chỉ định `cmap`, Matplotlib mặc định sử dụng colormap **`viridis`** (nếu cấu hình `image.cmap` chưa bị thay đổi).

Dải ánh xạ giá trị ở đây là:

`0 ─────────────────────────────── 15`  
`↓ ↓`  
`màu tối màu vàng sáng`

![Image](https://images.openai.com/static-rsc-4/MQwgXgGnWFDZ9st3bFiq_HjVunqPhI4jeaNyBbuctrRLTI2WJ5rCV-siD8VrFsaDTQkkLTuh3ByqcbyMZ2wKOGExBYSl-5PyHb3fupGwXNxjiSKOKLK9VbNfCeeShWyg8CEtWBJnhJ-t4oqdtwSAdMJezZOjsKIBtfr-mpd_dhA7maXL_89vZbXLVEJiiEIH?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/wvjLpxwwxsHSYayZlsYGCIno5phxNG8Mv8KsUZuutM5X-p5iZ4rGAoDSY19LHUAgYozyLguONZUUrS_1bFuWELzIqbrCpdDUxBUABc4arxqqAz3px7zkqrwpCr627BhIlT6pM9rXFFtWXCs1XzA8aXSNR3oaRmzPWk9W9q7BAr-4_zIgw4C0bfAH68oNybeb?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/RJ1Nf05A5_sA4XBQvuGHpSuv0pF4LVHf-JbyMDadl611zOIMqhkMu5gsUrpMiLABPQHiryF48sGOnYYo93dyC8kqqgKY4yBOHHhgoPv0gXnG5pTeg-ZPc1nOd1v-ZgImgwghwGLJLQJ0kJXO9osBobn0a6-MT3NW8DeV8CWjA8FUe-hMeVqYiYXkORzitP8Q?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/sYZcMJSdPCYerOwI1foYDaY3tLRUz58hEU4zK_jLnESVQWPoDrblL3hPhcMrj0O0IrJ2mHdAEpb6wHoyUXPl-m_ndWsOVipg0XxbpnkWPKXhn-xPWB_fDGTfhiwfSsFT6XnM3ioh6K46C9GyrRKotpjfgPcX1f0Ad_6D_-jiIhvv2T6Lds07saiPIfVdKVty?purpose=fullsize)

Trong đó:

- `vmin=0` → giá trị `0` ứng với đầu thấp nhất của colormap.
    
- `vmax=15` → giá trị `15` ứng với đầu cao nhất.
    
- Giá trị từ `0 → 15` được ánh xạ liên tục qua `viridis`.
    
- Giá trị `< 0` bị clip về màu tương ứng `0`.
    
- Giá trị `> 15` bị clip về màu tương ứng `15`.
    

Điểm quan trọng là **`vmin=0, vmax=15` không phải dải màu**, mà là **dải giá trị dữ liệu dùng để chuẩn hóa trước khi ánh xạ sang màu**.

Nếu bạn muốn **hiển thị giống hệt tác giả một cách tường minh**, có thể viết:

```python
plt.imshow(
    abs(radar_data[::20, ::20]),
    vmin=0,
    vmax=15,
    cmap="viridis",
    origin="lower"
)
```

Nếu bạn muốn, tôi có thể giải thích tiếp **tại sao tác giả lại chọn `vmin=0, vmax=15` cho `abs(radar_data)` và giá trị 15 này có ý nghĩa gì đối với dữ liệu Sentinel-1 L0**.