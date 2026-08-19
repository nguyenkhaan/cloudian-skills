
## 1. Workflow working with AI Agent 
### Step 1: Blackbox Exploring  

**Output:** `idea.md`: Document represent author's comprehensive idea about product 

In this step, we specialize in explore about our product's functionalities, based on comprehensive idea 

Brainstorming any idea about your project. Write all features you want. **Note**: Remember to find and try sample products currently in market. For example: If you do an eccommerce platform, you can reference some famous like: Shoppe or Etsy shopping. You can try and list their features. They are a valuable resources for your idea. 

Write document: `idea.md` with sections: Features, User Story (features users want in your product), System Story (features your system will satisfy)

Using AI skills: `interview-me` and `idea-refine` to enhance your idea into tight strong logic. 

Using AI Agent to rewrite `idea.md` into professional, detail and comprehensive document about your idea 

Reread the document to verify idea with AI writing document. Edit it if you see appororiate 

### Step 2: Architecture. 
- FIle idea.md: Chứa idea mô tả ban đầu về dự án 
- File prd.md: Mô tả yêu cầu của dự án, cũng như phân chia dự ra thành các module nhỏ. 
- File overplan.md: Mô tả dự án, tiến trình plan của dự án 
- File heatmap.md: Mô tả tầm ảnh hưởng của từng thành phần (module) đối với dự án 
- Khi prompt thì đề xuất thêm các giải pháp, rủi ro đối với AI Agent. 
- File api_spec.md: Một dạng file plan, nên sử dụng các dự án dựng BE thuần để cả team có thể nắm rõ được API này code như thế nào 
Các thành phần cốt lõi: 
- Kiểm tra cấu trúc dự án. Chỉnh sửa cấu trúc dự án 
- Thực hiện manual testing 
- Kiểm tra heatmap.md 
- Kiểm tra idea.md, tiến hành chỉnh sửa, bổ sung idea đểu phải do con người làm. 

Con người đóng vai trò hướng dẫn, hoặc có thể điều phối cho Agent Code. 

Kết hợp với các fil SKILL.md và các rule.md để agent có thể tự động hóa được trong công việc tối hơn. 

Thực hiện chỉnh sửa dự án: cloudian-skills: 
- Tạo thêm các SKILL mới:
+ SKILL để giúp cho Agent có thể thực hiện đánh giá nghiệp vụ công việc 
+ SKILL để giúp Agent hỏi các câu hỏi sau đó đề xuất ý tưởng cũng như công nghệ 
- Tạo thêm các rule mới: 
+ Rule yêu cầu Agent này đóng vai trò là một Agent làm Code 
+ Rule yêu cầu Agent này đóng vai trò là một Agent Tester: Xác định rõ phạm vi các file cần test, security có thể xảy ra, đề xuất giải pháp, rủi ro... 

