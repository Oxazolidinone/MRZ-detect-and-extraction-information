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

