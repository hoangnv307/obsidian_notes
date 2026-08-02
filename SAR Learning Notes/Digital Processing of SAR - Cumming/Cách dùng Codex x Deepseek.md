# Codex đề xuất: 
Có. Cách gọn nhất là dùng **custom subagent**: Codex lập kế hoạch và review, DeepSeek chỉ sửa code. Codex local hỗ trợ custom agents và custom model providers qua Responses API. ([Codex subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents), [custom providers](https://learn.chatgpt.com/docs/config-file/config-advanced#custom-model-providers))

Trong `~/.codex/config.toml`:

```toml
[agents]
enabled = true
max_concurrent_threads_per_session = 2

[model_providers.deepseek]
name = "DeepSeek"
base_url = "https://api.deepseek.com"
env_key = "DEEPSEEK_API_KEY"
wire_api = "responses"
```

Tạo `~/.codex/agents/deepseek_implementer.toml`:

```toml
name = "deepseek_implementer"
description = "Implements an approved plan; does not plan, review, or commit."
model = "deepseek-v4-flash"
model_provider = "deepseek"
sandbox_mode = "workspace-write"

developer_instructions = """
Read and follow AGENTS.md.
Implement only the plan supplied by the parent agent.
Make the smallest working diff.
Run the smallest relevant build or test.
Do not commit.
Return changed files, test results, and remaining concerns.
"""
```

Khai báo API key và restart Codex:

```bash
export DEEPSEEK_API_KEY="..."
codex
```

Sau đó prompt:

```text
Phân tích yêu cầu và tạo plan trước, chưa sửa code.
Sau khi plan hoàn chỉnh:
1. giao plan tuần tự cho deepseek_implementer;
2. chờ agent implement và chạy test;
3. tự review toàn bộ git diff về correctness, regression và missing tests;
4. nếu có lỗi, giao lại đúng finding cho deepseek_implementer sửa;
5. review lại và báo kết quả cuối cùng.
Không commit.
```

Hiện tại chỉ `deepseek-v4-flash` được tài liệu DeepSeek liệt kê là hỗ trợ Responses API; `deepseek-v4-pro` chưa hỗ trợ trực tiếp. ([DeepSeek model capabilities](https://api-docs.deepseek.com/quick_start/pricing))

Lưu ý: runtime của cuộc chat hiện tại chỉ cho mình spawn OpenAI subagents. Setup trên áp dụng cho **Codex local sau khi restart**. Nếu model DeepSeek bạn dùng không hỗ trợ Responses API, lúc đó mới cần MCP wrapper hoặc script orchestrator riêng.

## Sau khi bảo codex thực hiện
• Đã cấu hình:

  - /home/xiaoxin/.codex/config.toml
  - /home/xiaoxin/.codex/agents/deepseek_implementer.toml

  Bạn cần mở terminal và chạy:

  read -rsp "DeepSeek API key: " DEEPSEEK_API_KEY &&
  export DEEPSEEK_API_KEY &&
  echo &&
  codex -C /home/xiaoxin/code/sar_application

  Không gửi API key vào chat. Trong phiên Codex mới, chỉ cần nói:

  Lập plan, giao deepseek_implementer implement tuần tự, sau đó review git diff.