# คู่มือการทำงาน Application Support: Power BI (Part 3)

---

## 3. บทหัวข้อ (ต่อ)

### 3.8 Disaster Recovery (DR) และการสำรองข้อมูล (Backup Strategy)

**เรื่อง:** การวางแผนสำรองข้อมูล การกู้คืน Workspace/Dataset ที่ถูกลบ และการกู้คืนในกรณีที่ Region ล่ม

**วัตถุประสงค์ของหัวข้อนี้:** เพื่อให้ทีม Support ทราบขั้นตอนการกู้คืน (Recovery) เมื่อเกิดเหตุการณ์ข้อมูลสูญหายหรือระบบ Cloud ล่ม

**เป้าหมายของหัวข้อนี้:** สามารถกู้คืน Workspace หรือ Dataset ที่ถูกลบโดยอุบัติเหตุได้ภายใน 1 ชั่วโมง และมีแผนรับมือเมื่อ Cloud ระดับ Region ล่ม

**เนื้อหา / รายละเอียด:**
Power BI เป็น SaaS (Software as a Service) ทำให้ Microsoft ดูแล Infrastructure ให้ แต่สิ่งที่ทีม Support ต้องระวังคือ **Human Error** (เช่น Developer เผลอกดลบ Workspace, ลบ Dataset, หรือเขียน DAX ทับจนข้อมูลเพี้ยน) 

**แนวทางการสำรองข้อมูล (Backup) ใน Power BI:**
1.  **Workspace Recovery (Premium/Fabric):** หาก Workspace ถูกลบ และยังอยู่ในช่วง 30 วัน ทีม Admin สามารถกู้คืนได้ผ่าน Power BI Admin Portal โดยตรง
2.  **Version History (Git Integration):** หากใช้ Workspace ที่เชื่อมกับ Azure DevOps (Git) จะสามารถ Rollback กลับไปใช้ Version ก่อนหน้าได้ทันที
3.  **การดึงไฟล์ .PBIX สำรอง:** การ Backup ที่ดีที่สุดคือการ Export ไฟล์ `.pbix` (สำหรับ Semantic Model และ Report) ออกมาเก็บไว้นอกระบบ (On-premises หรือ Cloud Storage อื่น)
4.  **การสำรองข้อมูลด้วย XMLA Endpoint:** ใน Capacity ระดับ Premium ขึ้นไป สามารถใช้เครื่องมือภายนอก (เช่น Tabular Editor, SSMS) เชื่อมต่อผ่าน XMLA เพื่อทำการ Backup โครงสร้าง Metadata ได้

**การกู้คืนกรณี Region ล่ม (Disaster Recovery):**
Power BI มี Multi-AZ (Availability Zone) ในตัว แต่หากทั้ง Region ล่ม (เช่น Southeast Asia ล่ม) ข้อมูลจะไม่ถูก Replicate ข้าม Region โดยอัตโนมัติ ทีม Support ต้องแจ้ง Business ว่าไม่สามารถใช้งานได้จนกว่า Microsoft จะกู้คืน หรือต้องมีแผน Manual Failover ไปยัง Tenant สำรองที่ตั้งไว้ใน Region อื่น (ซึ่งมีค่าใช้จ่ายสูงมาก)

**สรุปย่อ:** การ Backup หลักของ Power BI คือการ Export ไฟล์ `.pbix` และการพึ่งพา Workspace Recovery ภายใน 30 วัน

**ประโยชน์ที่ได้รับ:** ป้องกันการสูญหายของ Report สำคัญ และมีหลักฐานอ้างอิงเมื่อเกิดข้อผิดพลาด

**ข้อควรระวัง:** การ Export ไฟล์ `.pbix` จาก Power BI Service อาจติดข้อจำกัดเรื่องขนาดไฟล์ (หาก Dataset ใหญ่เกินไป) และต้องมีสิทธิ์ระดับ Contributor ขึ้นไป

**ข้อดี:** กู้คืน Workspace ที่ถูกลบได้ง่ายภายใน 30 วัน
**ข้อเสีย:** การ Backup อัตโนมัติแบบ Full Model ทำได้ยากกว่า Database ทั่วไป (ต้องใช้ XMLA)
**ข้อห้าม:** ❌ ห้ามลบ Workspace ทิ้งโดยไม่แจ้งทีม เพราะจะทำให้ User ทุกคนเข้าใช้งานไม่ได้ทันที (แม้จะกู้คืนได้ แต่ใช้เวลา)

**ตัวอย่าง (การกู้คืน Workspace ที่ถูกลบ):**
```powershell
# ต้องใช้สิทธิ์ Power BI Administrator
# 1. ติดตั้งโมดูล MicrosoftPowerBIMgmt
Install-Module -Name MicrosoftPowerBIMgmt

# 2. Login
Connect-PowerBIServiceAccount

# 3. ค้นหา Workspace ที่ถูกลบ (Restorable)
Get-PowerBIWorkspace -Scope Organization -Include All -Restorable

# 4. กู้คืน Workspace
Restore-PowerBIWorkspace -Id "Workspace-ID-ที่ถูกลบ" -RestoredName "Workspace-ชื่อใหม่"
```

**สรุปย่อ เน้น เข้าใจ และจดจำ:**
> **Power BI ไม่มีปุ่ม Backup Database** → ต้อง Export `.pbix` หรือใช้ Git → ถ้าลบ Workspace ต้องกู้คืนภายใน **30 วัน**

**แหล่งที่มา:**
- Microsoft Learn, "Power BI implementation planning: Disaster recovery"
- EPC Group, "Power BI Backup and Disaster Recovery"

---

### 3.9 การจัดการสิทธิ์ License และการแก้ไขปัญหา User เข้าใช้งานไม่ได้ (Licensing & Access)

**เรื่อง:** การวินิจฉัยปัญหา User ไม่สามารถเข้าถึง Report หรือแจ้งว่า "ระบบบอกให้อัปเกรด License"

**วัตถุประสงค์ของหัวข้อนี้:** เพื่อให้ทีม Support (Level 1) สามารถตรวจสอบและแก้ไขปัญหาเรื่อง License และสิทธิ์การเข้าถึงเบื้องต้นได้

**เป้าหมายของหัวข้อนี้:** ลด Ticket ที่เกี่ยวกับการขอสิทธิ์ (Access Request) และแก้ไขได้ภายใน 15 นาที

**เนื้อหา / รายละเอียด:**
ปัญหา "User เข้าใช้งานไม่ได้" ใน Power BI มีสาเหตุหลัก 3 ประการ:
1.  **License ไม่เพียงพอ:** 
    *   **Free License:** ดูได้เฉพาะ Report ที่อยู่ใน Workspace ของตัวเอง หรือ Report ที่แชร์แบบ "Publish to Web" (ซึ่งไม่ปลอดภัย)
    *   **Pro License:** ดู Report ใน Shared Workspace ได้
    *   **PPU (Premium Per User):** ดู Report ที่อยู่ใน Premium Workspace ได้
2.  **สิทธิ์ใน Workspace ไม่ถูกต้อง:** User มี Pro License แต่ไม่ได้ถูกเพิ่มเข้าไปใน Workspace หรือ Power BI App นั้นๆ
3.  **Tenant Settings บล็อก:** แอดมินปิดการใช้งานฟีเจอร์บางอย่าง (เช่น ปิดการแชร์ภายนอกองค์กร, ปิดการ Export ข้อมูล)

**การวิเคราะห์และแนวทางแก้ไข:**
*   **ตรวจสอบ License:** เข้าไปที่ Microsoft 365 Admin Center → Users → Active Users → เลือก User → ดูที่แท็บ Licenses ว่ามี Power BI Pro หรือ PPU หรือไม่
*   **ตรวจสอบ Error Code:** ถ้า User แจ้ง Error "This content requires a Power BI Pro license" แปลว่า Workspace นั้นอยู่บน Shared Capacity และ User ไม่มี Pro License
*   **ตรวจสอบ App vs Workspace:** หาก User ถูกเพิ่มใน Workspace แล้วแต่ยังไม่เห็น ให้ตรวจสอบว่า Report ถูก Publish เป็น App หรือยัง และ User ได้รับสิทธิ์ใน App หรือไม่

**สรุปย่อ:** ปัญหาการเข้าถึง 90% มาจาก User ไม่มี Pro License หรือไม่ได้ถูกเพิ่มใน App

**ประโยชน์ที่ได้รับ:** ลดระยะเวลาการรอคอยของ User และลดภาระของทีม Admin

**ข้อควรระวัง:** การให้สิทธิ์ระดับ Admin ใน Workspace ควรทำเฉพาะ Developer เท่านั้น (User ทั่วไปควรเป็น Viewer)

**ข้อดี:** ตรวจสอบได้ง่ายผ่าน M365 Admin Center
**ข้อเสีย:** การขอซื้อ License เพิ่มต้องผ่านฝ่ายจัดซื้อ (ใช้เวลานาน)
**ข้อห้าม:** ❌ ห้ามแนะนำให้ User ใช้บัญชีส่วนตัว (Gmail, Hotmail) สมัคร Power BI เพื่อดูข้อมูลบริษัท (ผิดนโยบายความปลอดภัยอย่างร้ายแรง)

**ตัวอย่าง (Template การตอบกลับ Ticket เบื้องต้น):**
```text
สวัสดีคุณ [ชื่อ User],

ทีม Support ได้ตรวจสอบการเข้าใช้งาน Report [ชื่อ Report] แล้ว 
พบว่าบัญชีของคุณยังไม่ได้รับการอนุญาต (License) สำหรับการเข้าถึง Report นี้

เพื่อให้สามารถเข้าใช้งานได้ กรุณาดำเนินการดังนี้:
1. หากคุณใช้งานผ่าน App: กรุณาตรวจสอบว่าคุณได้รับเชิญ (Invitation) เข้า App แล้วหรือไม่
2. หากคุณใช้งานผ่าน Workspace: กรุณาแจ้งหัวหน้าทีมของคุณเพื่อขอสิทธิ์ (Access Request) ผ่านระบบ IT

หากคุณมี Power BI Pro License แล้ว แต่ยังไม่สามารถเข้าใช้งานได้ กรุณาแนบภาพหน้าจอ Error เพื่อให้ทีมเทคนิคตรวจสอบเพิ่มเติมครับ

ขอแสดงความนับถือ,
ทีม Application Support
```

**สรุปย่อ เน้น เข้าใจ และจดจำ:**
> **Pro License + Workspace/App Permission + Tenant Settings** = 3 สาเหตุหลักที่ User เข้าใช้งานไม่ได้

**แหล่งที่มา:**
- Microsoft Learn, "Power BI licensing for users"
- Power BI Consulting, "Power BI Licensing and Access Troubleshooting"

---

### 3.10 การตรวจสอบ Audit Logs และกิจกรรมการใช้งาน (Usage & Audit Monitoring)

**เรื่อง:** การใช้ Audit Log เพื่อตรวจสอบว่าใครทำอะไร ในกรณีเกิดปัญหา หรือเพื่อลบ Report ที่ไม่มีคนใช้งาน

**วัตถุประสงค์ของหัวข้อนี้:** เพื่อให้ทีม Support สามารถสืบสวน (Forensic) เมื่อเกิดเหตุการณ์ผิดปกติ เช่น Report ถูกลบ, ข้อมูลถูก Export ออกไป, หรือหาคนที่แก้ไข Dataset

**เป้าหมายของหัวข้อนี้:** สามารถดึง Log เพื่อหาสาเหตุและผู้รับผิดชอบได้ภายใน 24 ชั่วโมง

**เนื้อหา / รายละเอียด:**
Power BI มี Audit Log 2 ระดับ:
1.  **Activity Log (ระดับ Workspace):** ดูได้จากใน Power BI Service โดยตรง (ดูว่าใครเข้า Report ไหน, เมื่อไหร่)
2.  **Unified Audit Log (ระดับ Tenant):** ต้องดึงผ่าน Microsoft Purview (Compliance Portal) หรือใช้ PowerShell/API ซึ่งจะครอบคลุมทุก Workspace ในองค์กร

**กิจกรรมที่ควรตรวจสอบ (Key Activities):**
*   `ViewReport`: ใครดู Report บ้าง
*   `ExportReport` / `ExportData`: ใคร Export ข้อมูลออกจากระบบ (สำคัญมากสำหรับ Data Loss Prevention)
*   `DeleteDataset`: ใครลบ Dataset
*   `UpdateDatasetParameters`: ใครแก้ไข Connection String

**สรุปย่อ:** ใช้ Unified Audit Log (Purview) ในการสืบสวนเหตุการณ์ที่กระทบความปลอดภัยหรือการสูญหายของข้อมูล

**ประโยชน์ที่ได้รับ:** มีหลักฐานทางดิจิทัล (Digital Forensics), ตรวจสอบ Compliance ได้

**ข้อควรระวัง:** Audit Log ของ Microsoft 365 จะถูกเก็บไว้ 90 วัน (default) หากต้องการเก็บนานขึ้นต้องซื้อ Add-on เพิ่ม

**ข้อดี:** ตรวจสอบย้อนหลังได้เกือบทุก Action สำคัญ
**ข้อเสีย:** การ Query Log จำนวนมากต้องใช้ความรู้เรื่อง PowerShell หรือ KQL
**ข้อห้าม:** ❌ ห้ามใช้ Audit Log เพื่อสอดส่องการใช้งานส่วนตัวของพนักงานโดยไม่มีเหตุผลทางธุรกิจ (ผิดกฎหมาย PDPA)

**ตัวอย่าง (PowerShell ค้นหา Audit Log):**
```powershell
# ต้องเชื่อมต่อกับ Exchange Online / Purview
Connect-ExchangeOnline

# ค้นหาว่าใครลบ Dataset ในช่วง 7 วันที่ผ่านมา
Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) `
    -EndDate (Get-Date) `
    -Operations "DeleteDataset" `
    -ResultSize 100 | 
    Select-Object CreationDate, UserIds, Operations, AuditData |
    Format-List
```

**สรุปย่อ เน้น เข้าใจ และจดจำ:**
> **Unified Audit Log** เก็บ 90 วัน → ใช้ตรวจสอบ `ExportData` (ข้อมูลรั่ว) และ `DeleteDataset` (ข้อมูลหาย)

**แหล่งที่มา:**
- Microsoft Learn, "Use audit logs in Power BI"
- Microsoft Purview, "Search the audit log"

---

## 4. Workflow

### 4.1 PSR (Problem-Solution-Result) - Disaster Recovery
| หัวข้อ | คำอธิบาย | ตัวอย่าง |
| :--- | :--- | :--- |
| **Problem** | มี Report Dashboard ที่สำคัญถูกลบ และทีม Support ไม่สามารถกู้คืนได้เพราะเลย 30 วัน | "User แจ้งว่า Report ถูกเผลอลบ ไปเมื่อ 2 เดือนก่อน ต้องกู้คืนให้ได้" |
| **Solution** | ไม่สามารถใช้ Workspace Recovery ได้ → ทำการดึงไฟล์ `.pbix` Backup จาก Cloud Storage ที่ทีม Backup ไว้อาทิตย์ละครั้ง แล้วนำมา Publish ใหม่ พร้อมขอ Credential จาก Data Owner | "กู้คืนจากไฟล์ Backup สำรอง และแจ้งให้ User อัปเดต Connection" |
| **Result** | Report กลับมาใช้งานได้ แต่มีข้อมูลย้อนหลังถึงวันสุดท้ายที่ Backup ไว้ (เช่น อาทิตย์ที่แล้ว) | "กู้คืนสำเร็จ แต่ขาดข้อมูล 3 วัน ต้องแจ้ง Business ให้ทราบ" |

### 4.2 RCA (Root Cause Analysis) - Access Issue
1.  **Why?** ทำไม User ถึงเปิด Report ไม่ได้ (ขึ้น Error 401/403)?
    → เพราะ User ไม่ได้รับสิทธิ์ใน App
2.  **Why?** ทำไม User ถึงไม่ได้รับสิทธิ์?
    → เพราะ User ย้ายแผนกใหม่ และระบบไม่ได้ Sync สิทธิ์อัตโนมัติ
3.  **Why?** ทำไมระบบไม่ Sync สิทธิ์?
    → เพราะใช้การเพิ่มสิทธิ์ด้วยมือ (Manual) ไม่ได้ผูกกับ Azure AD Group
4.  **Why?** ทำไมถึงใช้ Manual?
    → **Root Cause:** ทีมไม่มีกระบวนการใช้ Azure AD Security Group ในการจัดการสิทธิ์ App
    **Action Item:** ปรับกระบวนการ Onboarding พนักงาน โดยใช้ AD Group Mapping กับ Power BI App

### 4.3 KPI SMART - Access & Recovery
| ตัวชี้วัด (KPI) | Specific | Measurable | Achievable | Relevant | Time-bound |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Time to Restore Access** | เวลาเฉลี่ยในการแก้ไขปัญหา User เข้าใช้งานไม่ได้ | < 15 นาที | ทีมมี Checklist ตรวจ License/Sิทธิ์ | ลดผลกระทบต่อการทำงาน | วัดผลรายเดือน |
| **Recovery Point Objective (RPO)** | ช่วงเวลาที่ยอมรับได้ของข้อมูลที่อาจสูญหาย | < 24 ชั่วโมง | มี Backup PBIX รายวัน | ควบคุมความเสียหาย | วัดผลรายไตรมาส |
| **Audit Log Review** | จำนวนครั้งที่ตรวจสอบ Audit Log | อย่างน้อย 1 ครั้ง/เดือน | ตั้ง Alert อัตโนมัติเมื่อ Export ข้อมูลมาก | ตรวจสอบ Compliance | วัดผลรายเดือน |

---

## 5. สรุป

### 5.1 ประโยชน์ที่ได้รับ
- **Operational Excellence:** ทีม Support มีกระบวนการกู้คืน (Recovery) และตรวจสอบ (Audit) ที่ชัดเจน ลดความตื่นตระหนกเมื่อเกิดปัญหา
- **Security & Compliance:** การตรวจสอบ Audit Log และการจัดการ License อย่างถูกต้องช่วยให้องค์กรปลอดภัยจากข้อมูลรั่วไหล
- **User Satisfaction:** การแก้ไขปัญหาเรื่องสิทธิ์ได้อย่างรวดเร็ว (ภายใน 15 นาที) สร้างความประทับใจให้กับ User

### 5.2 ข้อดี
- การใช้คำสั่ง PowerShell ในการดึงข้อมูล Audit Log และกู้คืน Workspace เป็นวิธีที่รวดเร็วและทรงพลัง
- มี Built-in Recovery สำหรับ Workspace ที่ถูกลบ (ภายใน 30 วัน) ทำให้อุ่นใจได้ระดับหนึ่ง

### 5.3 ข้อเสีย
- **ค่าใช้จ่ายในการ Backup:** หากต้องทำ Backup ไฟล์ `.pbix` ขนาดใหญ่มาก ๆ ต้องใช้พื้นที่จัดเก็บที่เพิ่มขึ้น
- **การสืบสวน (Forensics):** การ Query Unified Audit Log อาจใช้เวลานานหากดึงข้อมูลเป็นระยะเวลานาน ๆ

### 5.4 ข้อห้าม
- ❌ **ห้ามลบ Workspace ทิ้งโดยเด็ดขาด** — แม้จะกู้คืนได้ภายใน 30 วัน แต่จะกระทบกับ User ทันที และชื่อ App ID อาจเปลี่ยน
- ❌ **ห้ามใช้ Personal Email ในการแชร์ Report** — ผิดนโยบายความปลอดภัยขององค์กร
- ❌ **ห้ามให้สิทธิ์ Admin/Member แก่ User ทั่วไป** — ให้ใช้สิทธิ์ Viewer หรือแชร์ผ่าน App เท่านั้น
- ❌ **ห้ามเพิกเฉยต่อ Audit Log ที่แจ้งเตือนการ Export ข้อมูลจำนวนมาก** — อาจเป็นการขโมยข้อมูล (Data Exfiltration)

---
**เอกสารอ้างอิง:**
- Microsoft Learn: Power BI Licensing, Audit Logs, Disaster Recovery
- EPC Group: Power BI Enterprise Governance & Security
- Microsoft Purview: Unified Audit Log Documentation