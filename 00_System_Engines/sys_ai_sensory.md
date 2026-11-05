---
id: "sys:ai_sensory"
type: "core_engine_system"
vision_cone_base_deg: 120
hearing_radius_base_blocks: 16
blood_scent_radius_blocks: 40
subscribers: ["[[mob_quy_nhap_trang]]", "[[mob_ho_tinh]]", "[[mob_ma_tranh]]", "[[mob_tho_phi]]"]
---
# Cây Cảm Biến Giác Quan & Trí Tuệ Nhân Tạo (Sensory AI Engine)

## 1. Ba Kênh Giác Quan Sinh Học
- **Thị giác (Vision):** Nón nhìn $120^\circ$. Bị che khuất bởi sương mù [[env_xuan_am_nom]] hoặc bóng tối sâu trong [[zone_lang_mo_tang_1]].
- **Thính giác (Hearing):** Bán kính nghe phụ thuộc loại tiếng động:
  - Đi rón rén: `4 blocks`
  - Chạy nhanh: `16 blocks`
  - Tiếng đập rèn sắt [[bst_lo_ren_than_da]]: `60 blocks`
  - Tiếng nổ súng [[wpn_sung_hoa_mai]] hoặc [[wpn_ong_hoa_ho]]: `200 blocks`
- **Khứu giác Máu (Scent):** Dị thú đánh hơi thấy vết thương hở [[wnd_laceration]] hoặc [[wnd_bleeding_arterial]] từ khoảng cách lên tới `40 blocks`.
