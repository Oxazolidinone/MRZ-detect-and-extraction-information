# Hướng Dẫn Workflow ComfyUI

## Giới Thiệu
Để tiện lợi hơn trong việc cài đặt các model và custom node, bạn nên cài đặt **ComfyUI Manager**. Công cụ này giúp đơn giản hóa quá trình cài đặt và quản lý các thành phần cần thiết. Dưới đây là hướng dẫn về các node cơ bản, thông số, và các thiết lập cần thiết để xây dựng workflow từ đơn giản đến phức tạp trong ComfyUI.

---

## Workflow Cơ Bản
Mỗi workflow cơ bản thường bao gồm các thành phần sau:

### 1. Load Model Diffusion
- **Mục đích**: Tải model diffusion có sẵn để thực hiện quá trình sinh ảnh.
- **Output**: Các node Model, Clip, và VAE.

### 2. Text Prompt Node
- **Chức năng**: Nhận output từ model và cho phép nhập nội dung text để tạo ra hình ảnh.
- **Các Sub-node**:
  - **Positive**: Định rõ các yếu tố cần xuất hiện trong ảnh.
  - **Negative**: Định rõ các yếu tố không muốn có trong ảnh.
- **Kết nối**: Kết nối trực tiếp với **KSampler** để kiểm soát các thông số.

### 3. KSampler
- **Mục đích**: Lấy mẫu từ model, biến đổi noise ban đầu thành ảnh hoàn chỉnh dựa trên nội dung trong prompt.
- **Thông số chính**:
  - **Seed**: Tương tự như `random seed`, thiết lập tính ngẫu nhiên.
  - **Control After Generate**: Điều chỉnh hình ảnh sau khi đã tạo; các chế độ gồm `fixed`, `randomize`, `increment`, và `decrement`.
  - **Steps**: Số bước trong quá trình lấy mẫu. Số bước càng cao thì ảnh càng chi tiết, nhưng thời gian thực hiện cũng tăng lên.
  - **CFG**: Điều chỉnh mức độ mà model bám sát vào prompt so với các yếu tố ngẫu nhiên.
  - **Sample Name**: Phương pháp lấy mẫu, xác định cách thức model chuyển đổi noise thành ảnh hoàn chỉnh.
  - **Scheduler**: Thuật toán điều khiển cách giảm dần noise qua mỗi bước.
  - **Denoise**: Điều chỉnh cường độ giảm noise trong mỗi bước lấy mẫu, ảnh hưởng đến độ sắc nét và mượt mà của hình ảnh.

### 4. Node Empty Latent
- **Mục đích**: Điều chỉnh kích thước ảnh và số lượng ảnh muốn tạo ra mỗi lần chạy workflow.

### 5. Node VAE Decode
- **Chức năng**: Chuyển đổi các vector latents thành hình ảnh có thể nhìn thấy được.

### 6. Node Save Image
- **Mục đích**: Lưu trữ hình ảnh đã tạo để xem hoặc sử dụng sau.

---

## Workflow Chi Tiết

### Cấu Trúc Workflow
Một workflow từ cơ bản đến phức tạp bao gồm các thành phần sau:

- **Node Load Checkpoint**
  - **Output**: Model, Clip, VAE.
  
- **Node Clip Text Encode**
  - **Input**: Clip.
  - **Output**: Conditioning.
  
- **Node VAE Decode**
  - **Input**: Samples, VAE.
  - **Output**: Image.
  
- **Node KSampler**
  - **Input**: Model, Positive, Negative, Latent Image.
  - **Output**: Latent.
  
- **Node Empty Latent**
  - **Output**: Latent Image.
  
- **Node Save Image**
  - **Input**: Image.

---

### Tạo Ảnh Bằng Model Stable Diffusion Kết Hợp Với LoRA

Để workflow tạo ra được đối tượng cụ thể (ví dụ: một người cụ thể), bạn cần train một LoRA từ các mẫu ảnh chụp thực tế.

#### Cài Đặt Để Sử Dụng LoRA
- **Các thư viện cần**: `torch`, `diffusers`, `bitsandbytes`.
- **Model dùng để train**: Dreambooth cho Diffusers hoặc Kohya’s LoRA Trainer.

#### Huấn Luyện Từ Dataset
```bash
python Lora_ABC --dataset_path /path/to/dataset --output_dir /path/to/output --learning_rate 1e-4 --batch_size 4 --num_steps 10000 --rank 8
---

## Tạo Ảnh Từ Ảnh (Image to Image) Với Chế Độ Pose
Chế độ "Image to Image" cho phép bạn tạo ra một bức ảnh mới dựa trên dáng của một ảnh mục tiêu đầu vào. Dưới đây là hướng dẫn chi tiết về cách thiết lập workflow để thực hiện điều này.

### Các Node Cần Thiết
1. **Các Thành Phần Cơ Bản của Một Workflow**: Bao gồm các node cần thiết như Load Model, Clip Text Encode, VAE Decode, và KSampler.
   
2. **Thay Thế Node Empty Latent**:
   - **Node Load Image**: Để tải ảnh mục tiêu mà bạn muốn sử dụng làm cơ sở.
   - **Node VAE Encode**: Mã hóa ảnh đầu vào để chuyển đổi thành vector latents.
   - **Node Pose**: Lấy dáng của mục tiêu trong ảnh.

3. **Node Load ControlNet Model**: Tải mô hình ControlNet mà bạn muốn sử dụng để điều chỉnh quá trình tạo ảnh.

4. **Node Apply ControlNet**: Được chèn vào giữa flow của Clip Text Prompt và KSampler để điều chỉnh ảnh dựa trên dáng mục tiêu.

### Quy Trình Thiết Lập
- **Bước 1**: Tải mô hình diffusion và các node cần thiết thông qua ComfyUI Manager.
  
- **Bước 2**: Thiết lập các node trong workflow như sau:
   - **Load Image Node**: Nhập ảnh đầu vào bạn muốn sử dụng.
   - **VAE Encode Node**: Kết nối với Load Image Node để mã hóa ảnh.
   - **Pose Node**: Kết nối với VAE Encode Node để lấy dáng của ảnh mục tiêu.
   - **Load ControlNet Node**: Kết nối với Pose Node.
   - **Apply ControlNet Node**: Kết nối với Clip Text Prompt và KSampler.

### Các Thông Số Ảnh Hưởng Đến Đầu Ra
- **Strength Trong Node Apply ControlNet**: 
   - Thông số này điều chỉnh mức độ lắng nghe của model với dáng ảnh đầu vào. Nếu giá trị strength giảm (khoảng 0.4-0.6), model sẽ tuân theo prompt text tốt hơn. Nếu strength gần 1, model sẽ tạo ra ảnh giống ảnh gốc hơn là dựa vào prompt.

- **Denoise Trong KSampler**:
   - Điều chỉnh cường độ noise giảm trong quá trình sampling. Giá trị denoise quá lớn sẽ dẫn đến sáng tạo nhưng có thể sai lệch, trong khi giá trị quá nhỏ sẽ tạo ra ảnh giống với ảnh gốc. Giá trị khoảng 0.3 thường cho kết quả gần gũi với người thật hơn.

### Thay Thế Nếu Không Sử Dụng Được Node DWPose
Nếu bạn không thể sử dụng node DWPose (Node Pose), có thể thay thế bằng cách sử dụng OpenPose hoặc các phương pháp khác có sẵn trong ComfyUI để lấy dáng từ ảnh.

---

Hy vọng rằng hướng dẫn này sẽ giúp bạn thiết lập thành công quy trình tạo ảnh từ ảnh trong ComfyUI!

