## คู่มือการทำงาน Application Support: Power BI

**ผู้จัดทำ:** ทีม Application Support
**เวอร์ชัน:** 1.0
**สถานะ:** ต้นฉบับ

**ผู้แต่ง:** kongnakorn jantakun  
**อีเมล:** kongnakornjantakun@gmail.com  
**สถานะ:** ต้นฉบับ
**Reference:** https://github.com/kongnakornna/powerbi

---

## 1. บทนำ

### 1.1 วัตถุประสงค์
คู่มือเล่มนี้จัดทำขึ้นเพื่อเป็นแนวทางมาตรฐานสำหรับทีม Application Support ในการดูแลระบบ **Power BI** ตั้งแต่หลังการ Go-Live ครอบคลุมการติดตั้งและตั้งค่า (Configuration), การตรวจสอบและเฝ้าระวัง (Monitoring), การวิเคราะห์และแก้ไขปัญหา (Troubleshooting & RCA), ตลอดจนการบริหารจัดการ Incident อย่างเป็นระบบ เพื่อให้ระบบรายงานและ Dashboard ทางธุรกิจมีความพร้อมใช้งาน (Availability) และข้อมูลมีความถูกต้องเชื่อถือได้ (Data Trustworthiness) อยู่เสมอ

### 1.2 เป้าหมาย
1.  ลดระยะเวลา Mean Time to Detect (MTTD) และ Mean Time to Resolve (MTTR) เมื่อเกิดปัญหา Power BI
2.  สร้างมาตรฐานการทำงาน (Standard Operating Procedure) ที่ทีม Support สามารถปฏิบัติตามได้จริง
3.  ป้องกันปัญหาเชิงรุก (Proactive Support) เช่น การแจ้งเตือนก่อน Refresh ล้มเหลว หรือก่อน Capacity เต็ม
4.  มีเอกสารอ้างอิงสำหรับการติดตั้ง การตั้งค่า และ Template ต่าง ๆ ที่พร้อมนำไปใช้

---

## 2. บทนิยาม

| คำศัพท์ | ความหมาย |
| :--- | :--- |
| **Semantic Model (Dataset)** | โมเดลข้อมูลที่เก็บ Logical Layer (Tables, Relationships, Measures, DAX) ซึ่ง Report จะดึงข้อมูลไปแสดง |
| **On-premises Data Gateway** | ซอฟต์แวร์ที่ติดตั้งบน Windows Server ภายในองค์กร เพื่อเป็นสะพานเชื่อมต่อระหว่าง Power BI Service (Cloud) กับ Data Source (On-premises) |
| **Gateway Cluster** | การรวม Gateway หลาย Node เข้าด้วยกันเพื่อทำ High Availability (HA) และ Load Balancing |
| **Refresh (Import Mode)** | กระบวนการดึงข้อมูลจาก Source System เข้าสู่ Power BI Semantic Model ตาม Schedule |
| **DirectQuery** | โหมดการเชื่อมต่อที่ Power BI จะส่ง Query ไปยัง Source System แบบ Real-time ทุกครั้งที่มีการเรียกดู Report |
| **Capacity (Premium/Fabric)** | Resource (Compute & Memory) ที่จองไว้สำหรับประมวลผล Workload ของ Power BI โดยเฉพาะ |
| **DAX (Data Analysis Expressions)** | ภาษาสูตรที่ใช้ในการสร้าง Calculated Column, Calculated Table และ Measures |
| **Service Principal (SPN)** | Identity ที่ใช้สำหรับ Automation หรือ Application ในการเรียกใช้ Power BI REST API (เช่น การดึง Refresh History) |
| **PSR** | Problem-Solution-Result (Workflow การรายงานปัญหา) |
| **RCA** | Root Cause Analysis (การวิเคราะห์หาสาเหตุราก) |
| **KPI SMART** | Key Performance Indicator ที่ตั้งตามหลัก Specific, Measurable, Achievable, Relevant, Time-bound |

---

## 3. บทหัวข้อ

### 3.1 การติดตั้งและตั้งค่า On-premises Data Gateway (Enterprise Mode)

**เรื่อง:** การติดตั้ง Gateway สำหรับ Production เพื่อให้ Power BI Service สามารถเข้าถึง Data Source ภายในองค์กรได้อย่างเสถียรและมี HA

**วัตถุประสงค์ของหัวข้อนี้:** เพื่อให้ทีม Support สามารถติดตั้ง ตั้งค่า และสร้าง Cluster สำหรับ Gateway ได้อย่างถูกต้อง

**เป้าหมายของหัวข้อนี้:** Gateway พร้อมใช้งาน, มี Cluster อย่างน้อย 2 Node, และตั้งค่า Credential ถูกต้องตามหลัก Security

**เนื้อหา / รายละเอียด:**
On-premises Data Gateway เป็น Component สำคัญที่สุดในสถาปัตยกรรม Power BI แบบ Hybrid หาก Gateway ล่ม การ Refresh ทั้งหมดจะหยุดชะงัก และ DirectQuery Report จะแสดง Error ทันที สำหรับ Production แนะนำให้ใช้ **Standard Mode (Enterprise Gateway)** เท่านั้น ห้ามใช้ Personal Mode โดยเด็ดขาด เนื่องจาก Personal Mode ไม่รองรับ Cluster และผูกกับ User Account ซึ่งเสี่ยงต่อการหลุดเมื่อพนักงานลาออก

การติดตั้งควรทำบน Dedicated Server (Windows Server 2019+) ที่มีสเปกแนะนำ **8 Cores, 16 GB RAM, SSD** และควรอยู่ใน Datacenter เดียวกับ Data Source เพื่อลด Latency Gateway จะเป็นฝ่าย initiate connection ออกไปยัง Azure Service Bus ผ่าน HTTPS (Port 443) เท่านั้น ไม่จำเป็นต้องเปิด Inbound Firewall

**สรุปย่อ:** ติดตั้ง Standard Gateway บน Dedicated Server สเปกสูง, สร้าง Cluster อย่างน้อย 2 Node, และใช้ Service Account ในการรัน Service

**ประโยชน์ที่ได้รับ:** ระบบสามารถ Refresh ข้อมูลได้แม้ Node ใด Node หนึ่งล่ม (HA), ลดความเสี่ยงที่ Credential จะหมดอายุเพราะผูกกับ User

**ข้อควรระวัง:**
- ห้ามติดตั้ง Gateway บน Domain Controller หรือเครื่องที่มี Workload อื่นหนัก ๆ
- ต้อง Backup **Recovery Key** ของ Cluster เก็บไว้ใน Password Vault ทันที หากหายจะต้องกรอก Credential ใหม่ทั้งหมดด้วยมือ

**ข้อดี:** มี High Availability ในตัว, จัดการ Credential จากส่วนกลางได้
**ข้อเสีย:** ต้องมี Windows Server เฉพาะ, ต้องดูแล OS Patching เอง
**ข้อห้าม:** ❌ ห้ามใช้ Personal Gateway ใน Production เด็ดขาด

**ตัวอย่าง (ขั้นตอนการติดตั้งโดยย่อ):**
1.  ดาวน์โหลดและรัน Installer บน Server
2.  ล็อกอินด้วยบัญชีองค์กร
3.  เลือก **"Register a new gateway on this computer"**
4.  ตั้งชื่อ Gateway ให้สื่อความหมาย เช่น `GW-PROD-BKK-01`
5.  บันทึก Recovery Key
6.  ติดตั้ง Node ที่ 2 และเลือก **"Add to existing cluster"**

**สรุปย่อ เน้น เข้าใจ และจดจำ:**
> **Standard Gateway + Cluster 2 Node + Service Account + Backup Recovery Key** = รากฐานของ Power BI ที่เสถียร

**แหล่งที่มา:**
- Microsoft Learn, "Install an on-premises data gateway"
- EPC Group, "Power BI Gateway: Setup, Configuration, and Enterprise Best Practices"

---

### 3.2 การตั้งค่าและตรวจสอบ Capacity & Refresh Schedule

**เรื่อง:** การจัดการ Capacity (Premium/Fabric) และการตั้งเวลา Refresh ให้เหมาะสมกับ Workload

**วัตถุประสงค์ของหัวข้อนี้:** เพื่อให้ Support เข้าใจพฤติกรรมของ Capacity และสามารถตั้ง Schedule ที่ไม่ทำให้ระบบคอขวด

**เป้าหมายของหัวข้อนี้:** Capacity ทำงานไม่เกิน 80% และ Refresh Schedule กระจาย Load อย่างเหมาะสม

**เนื้อหา / รายละเอียด:**
หากองค์กรใช้ Power BI Premium หรือ Fabric Capacity การตรวจสอบ **Capacity Metrics App** เป็นสิ่งจำเป็น เนื่องจาก Capacity คือ Resource ที่ใช้ร่วมกัน (Shared) หาก Dataset ใดใช้ CPU หรือ Memory สูงเกินไป (Heavy Refresh) จะทำให้ Dataset อื่น ๆ บน Capacity เดียวกันช้าลงหรือ Refresh ไม่ทันเวลา

สำหรับ Refresh Schedule ควรหลีกเลี่ยงการตั้งเวลา Refresh ของ Dataset ขนาดใหญ่พร้อมกันในนาทีเดียวกัน (เช่น ทุกตัวตั้งไว้ 06:00 น.) ควรกระจายเวลาและคำนึงถึงเวลาเสร็จสิ้น (Completion Time) ที่ธุรกิจต้องการใช้ข้อมูล

**สรุปย่อ:** Capacity คือ Shared Resource, ต้อง Monitor ผ่าน Capacity Metrics App และกระจาย Refresh Schedule

**ประโยชน์ที่ได้รับ:** ป้องกัน Capacity Throttling และการแย่ง Resource กันเองระหว่าง Dataset

**ข้อควรระวัง:** การตั้ง Refresh ถี่เกินไป (เช่น ทุก 15 นาที) สำหรับ Dataset ขนาดใหญ่อาจทำให้ Capacity เต็มตลอดเวลา

**ข้อดี:** ควบคุมต้นทุน Capacity ได้, ระบบมีเสถียรภาพ
**ข้อเสีย:** ต้องใช้เวลาในการปรับจูน Schedule
**ข้อห้าม:** ❌ ห้ามตั้ง Refresh Schedule ของ Dataset ขนาดใหญ่ (>1GB) พร้อมกันทั้งหมด

**ตัวอย่าง (การตรวจสอบ Refresh History ผ่าน DAX):**
ใน Power BI Desktop สามารถใช้ DAX สร้างตารางประวัติ Refresh ได้ (สำหรับ Semantic Model ที่เปิด Auto Date/Time):
```dax
EVALUATE
SELECTCOLUMNS(
    'Date',
    "Refresh Time", [Date],
    "Table", "Model"
)
```
*(หมายเหตุ: วิธีที่แม่นยำที่สุดคือการใช้ REST API ตามหัวข้อ 3.3)*

**สรุปย่อ เน้น เข้าใจ และจดจำ:**
> **Capacity = Shared Resource** — กระจาย Refresh Load — Monitor ด้วย Capacity Metrics App

**แหล่งที่มา:**
- Microsoft Learn, Power BI Implementation Planning (Capacity Planning)
- team400.ai, "Supporting and Monitoring Power BI Content"

---

### 3.3 การวิเคราะห์ประสิทธิภาพ (Performance Troubleshooting) ด้วย Performance Analyzer และ DAX Studio

**เรื่อง:** การหาสาเหตุที่ Report หรือ Visual ทำงานช้า

**วัตถุประสงค์ของหัวข้อนี้:** เพื่อให้ Support สามารถแยกแยะได้ว่าปัญหาความช้ามาจาก DAX, การ Render Visual, หรือ Source System

**เป้าหมายของหัวข้อนี้:** ลดเวลาในการหาต้นเหตุ (Root Cause) ของ Performance Issue ภายใน 30 นาที

**เนื้อหา / รายละเอียด:**
เครื่องมือหลักที่ต้องใช้คือ **Performance Analyzer** (มีใน Power BI Desktop และ Service) เมื่อเริ่ม Record แล้วทำ Interaction กับ Report เครื่องมือจะแสดงเวลาที่ใช้แตกออกเป็น:
- **DAX Query:** เวลาที่ Visual ใช้ Query ข้อมูลจาก Model (มักเป็นสาเหตุหลัก)
- **Visual Display:** เวลาที่ใช้วาด Visual บนหน้าจอ
- **Direct Query:** เวลาที่ใช้ Query ไปยัง Source (ถ้าใช้โหมด DirectQuery)

หากพบว่า DAX Query ใช้เวลานาน ให้ใช้ **DAX Studio** เพื่อดู Server Timings ซึ่งจะแยกให้เห็นว่าเวลาส่วนใหญ่หมดไปกับ **Storage Engine (VertiPaq scan)** หรือ **Formula Engine (row-by-row iteration)**

**สรุปย่อ:** ใช้ Performance Analyzer แยกแยะปัญหา แล้วใช้ DAX Studio เจาะลึกที่ DAX Query

**ประโยชน์ที่ได้รับ:** สามารถบอก Developer ได้อย่างเจาะจงว่าต้อง Optimize ที่จุดไหน

**ข้อควรระวัง:** ต้อง **Clear Cache** ก่อนวัดผลทุกครั้ง เพื่อไม่ให้ผลลัพธ์ถูกบิดเบือนจาก Caching

**ข้อดี:** เป็น Built-in Tool ที่ใช้งานฟรีและแม่นยำ
**ข้อเสีย:** ต้องใช้ความรู้เรื่อง DAX และ Engine Architecture พอสมควร
**ข้อห้าม:** ❌ ห้ามสรุปว่าปัญหาอยู่ที่ DAX โดยไม่ดู Visual Display Time (บางครั้ง Visual ที่แสดงข้อมูลเยอะเกินไปก็ทำให้ช้าได้)

**ตัวอย่าง (การตีความ Timing):**
| Metric | Time (ms) | การตีความ |
| :--- | :--- | :--- |
| DAX Query | 2,500 | **ปัญหาหลักอยู่ที่ DAX/Model** |
| Visual Display | 150 | ปกติ |
| Other | 50 | ปกติ |

**สรุปย่อ เน้น เข้าใจ และจดจำ:**
> **Performance Analyzer** บอกว่าช้าตรงไหน → **DAX Studio** บอกว่าทำไม DAX ถึงช้า (Storage vs Formula Engine)

**แหล่งที่มา:**
- Microsoft Learn, "Use Performance analyzer to Diagnose Issues"
- EPC Group, "Power BI Performance Optimization"

---

### 3.4 การตรวจสอบและแจ้งเตือนความล้มเหลวของ Refresh ด้วย REST API และ Python (Monitoring Automation)

**เรื่อง:** การสร้างระบบ Monitoring อัตโนมัติสำหรับ Dataset Refresh Failure แทนการรอ Email จาก Power BI

**วัตถุประสงค์ของหัวข้อนี้:** เพื่อให้ Support ทราบถึงปัญหาก่อนที่ User จะแจ้ง

**เป้าหมายของหัวข้อนี้:** มี Script ที่รันเป็นรอบ (เช่น ทุก 30 นาที) เพื่อตรวจสอบและแจ้งเตือนเมื่อ Refresh ล้มเหลว

**เนื้อหา / รายละเอียด:**
แม้ว่า Power BI จะมีฟีเจอร์แจ้งเตือนเมื่อ Refresh ล้มเหลวทาง Email แต่ในองค์กรขนาดใหญ่ การรวม Alert ไว้ที่ศูนย์กลาง (เช่น ส่งเข้า Teams, Slack, หรือ Prometheus Alertmanager) จะทำให้ทีม Support จัดการได้เร็วขึ้น วิธีที่แนะนำคือใช้ **Power BI REST API** ร่วมกับ **Service Principal (SPN)** ในการดึงประวัติการ Refresh

**สรุปย่อ:** ใช้ SPN + REST API ดึง Refresh History มาตรวจสอบ แล้วส่ง Alert เข้าระบบ Monitoring กลาง

**ประโยชน์ที่ได้รับ:** ลดโอกาสที่ข้อมูลจะเก่า (Stale Data) โดยไม่มีใครรู้, มี Log เป็นหลักฐาน

**ข้อควรระวัง:** ต้องให้สิทธิ์ SPN ในระดับ Admin (Power BI Administrator) หรืออย่างน้อยต้องเป็น Member ของ Workspace

**ข้อดี:** ยืดหยุ่นสูง, สามารถ Retry อัตโนมัติได้
**ข้อเสีย:** ต้องเขียน Code และดูแล Script เอง
**ข้อห้าม:** ❌ ห้ามใช้ User Account ส่วนตัวในการรัน Automation (เพราะจะพังเมื่อคนนั้นลาออก)

**ตัวอย่าง (Python Code Template สำหรับตรวจสอบ Refresh):**
```python
import requests
import msal

# 1. Authentication ด้วย Service Principal
def get_access_token(tenant_id, client_id, client_secret):
    app = msal.ConfidentialClientApplication(
        client_id, authority=f"https://login.microsoftonline.com/{tenant_id}",
        client_credential=client_secret
    )
    result = app.acquire_token_for_client(scopes=["https://analysis.windows.net/powerbi/api/.default"])
    return result['access_token']

# 2. ดึงประวัติ Refresh ของ Dataset
def check_refresh_status(group_id, dataset_id, token):
    url = f"https://api.powerbi.com/v1.0/myorg/groups/{group_id}/datasets/{dataset_id}/refreshes?$top=1"
    headers = {"Authorization": f"Bearer {token}"}
    response = requests.get(url, headers=headers)
    data = response.json()
    if data.get('value'):
        latest = data['value'][0]
        return latest['status'], latest.get('serviceExceptionJson', '')
    return "Unknown", ""

# 3. Logic การแจ้งเตือน
def main():
    token = get_access_token("YOUR_TENANT_ID", "YOUR_CLIENT_ID", "YOUR_SECRET")
    # ตัวอย่าง Dataset ID
    status, error = check_refresh_status("YOUR_GROUP_ID", "YOUR_DATASET_ID", token)
    if status == "Failed":
        print(f"ALERT: Refresh Failed! Error: {error}")
        # ส่งเข้า Teams Webhook หรือระบบอื่น ๆ
    else:
        print(f"Status: {status}")

if __name__ == "__main__":
    main()
```

**สรุปย่อ เน้น เข้าใจ และจดจำ:**
> **SPN + REST API `/refreshes`** = หัวใจของการทำ Automated Refresh Monitoring

**แหล่งที่มา:**
- Medium (Vignesh Anandarangan), "Automating Power BI Dataset Refresh Failures using Python and REST API"
- Edureka Community, "Monitor Power BI dataset refresh failures"

---

## 4. Workflow

### 4.1 PSR (Problem-Solution-Result)
ใช้สำหรับการรายงาน Incident ที่เกี่ยวกับ Power BI เพื่อให้ทีมและผู้บริหารเข้าใจตรงกัน

| หัวข้อ | คำอธิบาย | ตัวอย่าง (Power BI) |
| :--- | :--- | :--- |
| **Problem** | อาการที่เกิดขึ้น, ผลกระทบต่อธุรกิจ | "Dashboard ยอดขายแสดงข้อมูลเก่า (เมื่อวาน) ตั้งแต่เวลา 09:00 น. กระทบทีม Sales ที่ต้องใช้ข้อมูลตัดสินใจ" |
| **Solution** | สิ่งที่ทีม Support เข้าไปดำเนินการ | "ตรวจพบว่า Gateway Node หลัก Offline จึงทำการ Failover ไปยัง Node สำรอง และ Trigger Manual Refresh ใหม่" |
| **Result** | ผลลัพธ์หลังการแก้ไข | "ข้อมูลกลับมาอัปเดตภายใน 15 นาที, ทีม Sales ใช้งานได้ตามปกติ, ไม่มี SLA ข้อใดถูกกระทบ" |

### 4.2 RCA (Root Cause Analysis) — 5 Whys
ใช้เมื่อเกิด Incident ซ้ำ ๆ หรือรุนแรง เพื่อหาสาเหตุรากที่แท้จริง

1.  **Why?** ทำไม Dashboard ถึงแสดงข้อมูลเก่า?
    → เพราะ Dataset Refresh ล้มเหลว
2.  **Why?** ทำไม Refresh ถึงล้มเหลว?
    → เพราะ Gateway ไม่สามารถเชื่อมต่อกับ SQL Server ได้
3.  **Why?** ทำไม Gateway ถึงเชื่อมต่อ SQL ไม่ได้?
    → เพราะ Credential ที่ใช้เชื่อมต่อหมดอายุ
4.  **Why?** ทำไม Credential ถึงหมดอายุ?
    → เพราะบัญชีที่ใช้เป็นของพนักงานที่เพิ่งลาออก และรหัสผ่านถูก Force เปลี่ยนตามนโยบาย
5.  **Why?** ทำไมถึงใช้บัญชีพนักงานในการเชื่อมต่อ?
    → **Root Cause:** ขาดมาตรฐานการใช้งาน Service Account สำหรับ Data Connection

**Action Item:** เปลี่ยน Credential ทั้งหมดใน Gateway ให้เป็น Service Account และเพิ่มขั้นตอนการตรวจสอบอายุ Credential ใน Maintenance Calendar

### 4.3 KPI SMART
การตั้งตัวชี้วัดสำหรับงาน Application Support Power BI

| ตัวชี้วัด (KPI) | Specific | Measurable | Achievable | Relevant | Time-bound |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Refresh Success Rate** | อัตราความสำเร็จของ Scheduled Refresh | ≥ 99% | ตั้งเป้าจาก Baseline ปัจจุบัน (เช่น 95%) | ส่งผลโดยตรงต่อความน่าเชื่อถือของข้อมูล | วัดผลรายเดือน |
| **MTTR (Mean Time to Resolve)** | เวลาเฉลี่ยในการแก้ไขปัญหา Power BI | < 2 ชั่วโมง | ทีมมี Runbook และเครื่องมือพร้อม | ลดผลกระทบต่อธุรกิจ | วัดผลรายไตรมาส |
| **Gateway Availability** | % เวลาที่ Gateway Cluster พร้อมใช้งาน | ≥ 99.9% | ใช้ Cluster 2 Node ขึ้นไป | เป็นโครงสร้างพื้นฐานหลัก | วัดผลรายเดือน |
| **Report Load Time (P95)** | เวลาที่ Report ใช้ในการโหลด (95th Percentile) | < 5 วินาที | Optimize DAX และ Model | กระทบ User Experience โดยตรง | วัดผลรายสัปดาห์ |

---

## 5. สรุป

### 5.1 ประโยชน์ที่ได้รับ
- **ความน่าเชื่อถือของข้อมูล:** การ Monitoring Refresh และ Gateway อย่างเป็นระบบช่วยให้มั่นใจได้ว่า Dashboard ที่ผู้บริหารใช้ตัดสินใจมีข้อมูลที่เป็นปัจจุบัน
- **ลดเวลาการแก้ไขปัญหา:** การมี Runbook และเครื่องมืออย่าง Performance Analyzer/DAX Studio ทำให้ทีม Support รู้จุดที่ต้องแก้ทันที ไม่ต้องเดา
- **มาตรฐานการทำงาน:** การใช้ PSR และ RCA ทำให้การสื่อสารภายในทีมและกับผู้บริหารเป็นไปในทิศทางเดียวกัน

### 5.2 ข้อดี
- Power BI มี Ecosystem ของเครื่องมือ Monitoring และ API ที่สมบูรณ์ ทำให้สามารถ Integrate เข้ากับระบบ Observability อื่น ๆ (เช่น Prometheus/Grafana) ได้
- รองรับ High Availability ผ่าน Gateway Cluster และ Capacity ที่มี Built-in Metrics

### 5.3 ข้อเสีย
- **ความซับซ้อนของ Gateway:** การดูแล Windows Server และ Credential เป็นงานที่ต้องใช้ทักษะเฉพาะ
- **ค่าใช้จ่าย Capacity:** หาก Workload เยอะ การใช้ Premium/Fabric Capacity มีค่าใช้จ่ายสูง
- **DAX Learning Curve:** การ Optimize DAX ต้องใช้เวลาเรียนรู้และประสบการณ์

### 5.4 ข้อห้าม
- ❌ **ห้ามใช้ Personal Gateway ใน Production** — เสี่ยงต่อ Single Point of Failure และ Credential หลุด
- ❌ **ห้ามใช้บัญชีพนักงาน (User Account) สำหรับ Data Connection หรือ Automation** — ให้ใช้ Service Account หรือ Service Principal เท่านั้น
- ❌ **ห้ามละเลยการ Backup Recovery Key ของ Gateway Cluster** — หากหายจะกู้คืนยากมาก
- ❌ **ห้ามตั้ง Refresh Schedule ของ Dataset ขนาดใหญ่พร้อมกันทั้งหมด** — จะทำให้ Capacity เกิด Throttling
- ❌ **ห้ามแก้ปัญหา Performance โดยการเพิ่ม Capacity อย่างเดียว** — ต้องแก้ที่ DAX/Model ก่อนเสมอ

---
**เอกสารอ้างอิง:**
- Microsoft Learn: Power BI Implementation Planning, Gateway Installation, Performance Analyzer
- team400.ai: Supporting and Monitoring Power BI Content, Tenant-Level Monitoring
- EPC Group: Power BI Gateway Enterprise Setup, Performance Optimization
- Power BI Consulting: Gateway Architecture for Large Enterprises


เพื่อความสมบูรณ์ของคู่มือ **Application Support สำหรับ Power BI** ผมขออนุญาตดำเนินการเขียนเนื้อหาต่อใน **บทหัวข้อที่เหลือ (Part 2)** ซึ่งครอบคลุมประเด็นที่ทีม Support เจอบ่อยที่สุดในระดับ Enterprise ได้แก่ **การจัดการความปลอดภัย (RLS)**, **การบริหารจัดการ Workspace และ Deployment Pipeline**, และ **การจัดการ Dataflow** ครับ

---

# คู่มือการทำงาน Application Support: Power BI (Part 2)
*(ต่อจาก 3.1 - 3.4)*

---

## 3. บทหัวข้อ (ต่อ)

### 3.5 การจัดการและการแก้ไขปัญหา Row-Level Security (RLS)

**เรื่อง:** การตั้งค่าและการแก้ไขปัญหาเมื่อผู้ใช้ไม่สามารถมองเห็นข้อมูล หรือมองเห็นข้อมูลเกินสิทธิ์ (Data Leakage)

**วัตถุประสงค์ของหัวข้อนี้:** เพื่อให้ทีม Support สามารถวินิจฉัยปัญหา RLS ที่เกิดจาก DAX, การ Mapping สิทธิ์ใน Workspace, หรือการตั้งค่า App

**เป้าหมายของหัวข้อนี้:** สามารถแก้ไขปัญหา User แจ้ง "เห็นข้อมูลไม่ครบ" หรือ "เห็นข้อมูลของแผนกอื่น" ได้ภายใน 1 ชั่วโมง

**เนื้อหา / รายละเอียด:**
RLS (Row-Level Security) ใน Power BI ทำงาน 2 ระดับ:
1. **Static RLS:** กำหนด Role ใน Power BI Desktop และระบุ User/Group เข้าไปใน Role นั้นผ่าน Power BI Service
2. **Dynamic RLS:** ใช้ DAX ฟังก์ชัน `USERPRINCIPALNAME()` เพื่อกรองข้อมูลตาม Login ของผู้ใช้ (แนะนำสำหรับองค์กรที่มีพนักงานหมุนเวียนบ่อย)

**สาเหตุที่พบบ่อยที่สุดที่ทำให้ RLS ผิดพลาด:**
- **User อยู่ใน Group ซ้อน Group:** Power BI Service ไม่รองรับ Nested Security Groups ในบางกรณี (เช่น User อยู่ใน Group A ซึ่งเป็นสมาชิกของ Group B ที่ถูกใส่ใน Role) ทำให้ User มองไม่เห็นข้อมูล
- **การแชร์ App vs Workspace:** หากแชร์ผ่าน Workspace โดยตรง สิทธิ์ RLS จะไม่ทำงานหาก User มีสิทธิ์ระดับ Admin/Member (จะเห็นข้อมูลทั้งหมด) ต้องบังคับให้ User เข้าใช้งานผ่าน **Power BI App** เท่านั้น
- **DAX ผิดพลาด:** เช่น การใช้ `USERNAME()` แทน `USERPRINCIPALNAME()` (USERNAME คืนค่าเป็น Domain\Username ซึ่งไม่ตรงกับ Email ใน Azure AD)

**สรุปย่อ:** ปัญหา RLS มักมาจากการ Mapping สิทธิ์ (Nested Group) หรือการเข้าถึงผ่าน Workspace แทน App

**ประโยชน์ที่ได้รับ:** ป้องกัน Data Leakage (ข้อมูลรั่วไหลข้ามแผนก) และลด Ticket ที่แจ้งว่า "ข้อมูลไม่ขึ้น"

**ข้อควรระวัง:** การทดสอบ RLS ต้องทำผ่าน "View as Role" ใน Desktop และ "Test as Role" ใน Service เท่านั้น ห้ามใช้การ Login ด้วยบัญชีอื่นมาเดา

**ข้อดี:** ปลอดภัยสูง, จัดการจากศูนย์กลาง
**ข้อเสีย:** Debug ยากหากมี Role ซับซ้อน
**ข้อห้าม:** ❌ ห้ามให้สิทธิ์ระดับ Member/Admin กับ User ทั่วไปใน Workspace (จะทะลุ RLS)

**ตัวอย่าง (Dynamic RLS Template):**
```dax
// ตาราง Security_Matrix ที่มีคอลัมน์ UserEmail และ Region
// สร้าง Role ชื่อ "Region_Manager"
[Region] = 
VAR CurrentUser = USERPRINCIPALNAME()
VAR UserRegion = 
    LOOKUPVALUE(
        Security_Matrix[Region],
        Security_Matrix[UserEmail], CurrentUser
    )
RETURN
    IF(ISBLANK(UserRegion), FALSE(), [Region] = UserRegion)
```

**ตัวอย่าง (REST API สำหรับตรวจสอบ RLS ผ่าน Python):**
```python
import requests

def get_dataset_users(group_id, dataset_id, token):
    url = f"https://api.powerbi.com/v1.0/myorg/groups/{group_id}/datasets/{dataset_id}/users"
    headers = {"Authorization": f"Bearer {token}"}
    response = requests.get(url, headers=headers)
    # คืนค่ารายชื่อ User และสิทธิ์ (Read, ReadReshare, Build)
    return response.json()

def check_rls_roles(group_id, dataset_id, token):
    url = f"https://api.powerbi.com/v1.0/myorg/groups/{group_id}/datasets/{dataset_id}/roles"
    headers = {"Authorization": f"Bearer {token}"}
    response = requests.get(url, headers=headers)
    # คืนค่า Role ทั้งหมดที่ตั้งไว้ใน Dataset
    return response.json()
```

**สรุปย่อ เน้น เข้าใจ และจดจำ:**
> **แชร์ผ่าน App เท่านั้น + หลีกเลี่ยง Nested Groups + ใช้ `USERPRINCIPALNAME()`** = RLS ทำงานถูกต้อง 100%

**แหล่งที่มา:**
- Microsoft Learn, "Row-level security (RLS) with Power BI"
- EPC Group, "Power BI Row-Level Security Best Practices"
- DAX Patterns, "Security Pattern"

---

### 3.6 การจัดการ Deployment Pipeline และ Workspace Governance

**เรื่อง:** การบริหารจัดการ Workspace (Dev → Test → Prod) และการย้าย Report ระหว่าง Environment

**วัตถุประสงค์ของหัวข้อนี้:** เพื่อให้ Support เข้าใจกระบวนการ Release และแก้ไขปัญหาเมื่อ Report พังหลังการ Deploy

**เป้าหมายของหัวข้อนี้:** ไม่มี Downtime ระหว่างการ Deploy และสามารถ Rollback ได้ทันที

**เนื้อหา / รายละเอียด:**
ในองค์กรระดับ Enterprise การแก้ไข Report บน Production โดยตรงถือเป็นเรื่องต้องห้าม (Anti-pattern) ต้องใช้ **Power BI Deployment Pipelines** โดยมี 3 Stage หลัก:
1. **Development:** ทีม Developer แก้ไข
2. **Test (UAT):** ทีม Business ตรวจสอบ
3. **Production:** ทีม Support ดูแล

**ปัญหาที่ทีม Support เจอบ่อย (Pipeline Issues):**
- **Data Source Rules ไม่ทำงาน:** เมื่อ Deploy จาก Test ไป Prod ระบบอาจยังชี้ไปที่ Database ของ Test ทำให้ข้อมูลบน Prod เป็นข้อมูล Test (หรือเลวร้ายกว่านั้นคือ เขียนทับข้อมูลจริง) ต้องตั้งค่า **Deployment Rules** ให้ถูกต้อง
- **Parameter Override:** ค่า Parameter เช่น URL ของ API หรือชื่อ Server ต้องถูก Override ในระดับ Stage ไม่ใช่ Hardcode ใน Report
- **Broken Permissions:** เมื่อ Deploy ใหม่ Workspace ID จะเปลี่ยน (ถ้าแชร์ผ่าน Workspace) แต่ถ้าแชร์ผ่าน App, App ID จะคงเดิม ทำให้ User ไม่ต้องเปลี่ยน Link

**สรุปย่อ:** ใช้ Deployment Pipeline 3 Stage และตั้งค่า Data Source Rules ให้ถูกต้องทุกครั้งก่อน Deploy

**ประโยชน์ที่ได้รับ:** ลดความเสี่ยงที่ Report พังบน Production, มีประวัติการ Deploy (Version History)

**ข้อควรระวัง:** หลังการ Deploy ต้องตรวจสอบว่า **Refresh Schedule** ถูกพกพาไปด้วยหรือไม่ (บางครั้ง Schedule ไม่ย้ายตามไปด้วย)

**ข้อดี:** ป้องกัน Human Error, แยกสภาพแวดล้อมชัดเจน
**ข้อเสีย:** ต้องมี Capacity ระดับ Premium/Fabric ขึ้นไปถึงจะใช้ฟีเจอร์นี้ได้
**ข้อห้าม:** ❌ ห้ามแก้ไข Report บน Workspace Production โดยตรง (ห้ามใช้ปุ่ม "Edit" บน Prod)

**ตัวอย่าง (ขั้นตอนการ Deploy ที่ปลอดภัย):**
1.  Developer อัปเดต Report ใน Dev Workspace
2.  Support ทำการ Deploy ไปยัง Test Workspace (กด Deploy)
3.  Support ตรวจสอบ **Deployment Rules** ใน Stage Test ว่าชี้ไปที่ Database Test
4.  Business ทำ UAT
5.  Support Deploy ไปยัง Production
6.  ตรวจสอบ **Deployment Rules** ใน Stage Prod ว่าชี้ไปที่ Database Production
7.  Trigger Manual Refresh ทันที
8.  ตรวจสอบ Report หน้า Dashboard ว่าทำงานปกติ

**สรุปย่อ เน้น เข้าใจ และจดจำ:**
> **Dev → Test → Prod** (ห้ามข้าม) + **ตรวจสอบ Deployment Rules** ทุกครั้ง = Zero Downtime Deployment

**แหล่งที่มา:**
- Microsoft Learn, "Introduction to deployment pipelines"
- team400.ai, "Managing Power BI Workspaces and Deployment Pipelines"

---

### 3.7 การจัดการ Dataflow และการแก้ไขปัญหาความล้มเหลวของ Dataflow

**เรื่อง:** การดูแล Dataflow (ETL บน Cloud) ซึ่งเป็น Layer ที่อยู่หน้า Dataset

**วัตถุประสงค์ของหัวข้อนี้:** เพื่อให้ Support เข้าใจความสัมพันธ์ระหว่าง Dataflow และ Dataset และแก้ไขปัญหาเมื่อ Dataflow Refresh ล้มเหลว

**เป้าหมายของหัวข้อนี้:** Dataset ที่ดึงข้อมูลจาก Dataflow ได้รับข้อมูลที่ถูกต้องและตรงเวลา

**เนื้อหา / รายละเอียด:**
Dataflow คือการทำ ETL (Extract, Transform, Load) ด้วย Power Query บน Power BI Service โดยเก็บผลลัพธ์ไว้ใน Azure Data Lake Storage (Gen2) เปรียบเสมือนเป็น "Shared Table" ที่ Dataset หลายตัวสามารถดึงไปใช้ได้

**สถาปัตยกรรมที่ควรรู้:**
`Source System` → `Dataflow (Refresh)` → `Data Lake` → `Dataset (Refresh)` → `Report`
*(หาก Dataflow ล้มเหลว Dataset จะไม่สามารถ Refresh ได้เลย)*

**การวิเคราะห์และแนวทางแก้ไข (Troubleshooting):**
- **Dataflow Refresh Timeout:** Dataflow มี Timeout 48 ชั่วโมง แต่หาก Source System ช้า แนะนำให้แบ่ง Dataflow เป็นส่วนย่อย ๆ (Chunking) หรือใช้ Enhanced Compute Engine
- **Linked Entities:** หาก Dataset ดึงจาก Linked Dataflow การ Refresh ต้องเริ่มที่ Dataflow ก่อนเสมอ
- **Credential หมดอายุ:** Dataflow ใช้ Credential แยกจาก Dataset ต้องเข้าไปอัปเดตใน Settings ของ Dataflow

**สรุปย่อ:** Dataflow ทำงานก่อน Dataset เสมอ หาก Dataflow พัง Dataset จะพังตาม

**ประโยชน์ที่ได้รับ:** ลดภาระ Source System (Extract ครั้งเดียวใช้หลาย Dataset), มาตรฐาน Transformation เดียวกัน

**ข้อควรระวัง:** การแก้ไข Dataflow จะส่งผลกระทบต่อ **ทุก Dataset** ที่ดึงข้อมูลไปใช้ (Impact Analysis ก่อนแก้เสมอ)

**ข้อดี:** Centralized Transformation, ลดขนาด Dataset
**ข้อเสีย:** Debug ยากกว่า Power Query ใน Desktop, ต้องมี Capacity Premium
**ข้อห้าม:** ❌ ห้ามลบ Column ใน Dataflow โดยไม่แจ้งเจ้าของ Dataset ที่เกี่ยวข้อง

**ตัวอย่าง (การตรวจสอบสถานะ Dataflow ผ่าน REST API):**
```python
def get_dataflow_refresh_history(group_id, dataflow_id, token):
    url = f"https://api.powerbi.com/v1.0/myorg/groups/{group_id}/dataflows/{dataflow_id}/transactions"
    headers = {"Authorization": f"Bearer {token}"}
    response = requests.get(url, headers=headers)
    # ดึงประวัติการ Refresh ของ Dataflow
    return response.json()
```

**สรุปย่อ เน้น เข้าใจ และจดจำ:**
> **Dataflow = ETL Layer** → ต้อง Refresh ก่อน Dataset เสมอ → ระวัง Impact ก่อนแก้ไข

**แหล่งที่มา:**
- Microsoft Learn, "Introduction to dataflows and self-service data prep"
- Power BI Consulting, "Dataflows Architecture and Best Practices"

---

## 4. Workflow (Specific to Advanced Topics)

### 4.1 PSR (Problem-Solution-Result) - RLS & Deployment
| หัวข้อ | คำอธิบาย | ตัวอย่าง |
| :--- | :--- | :--- |
| **Problem** | ผู้จัดการแผนกเหนือแจ้งว่าเห็นข้อมูลยอดขายของแผนกใต้ (Data Leakage) | "User แจ้งว่าเห็นข้อมูลผิดแผนก กระทบความลับทางธุรกิจ" |
| **Solution** | ตรวจสอบพบว่า User ถูกเพิ่มเข้า Workspace ในสิทธิ์ Member ทำให้ RLS ไม่ทำงาน แก้ไขโดยการถอดสิทธิ์ Member และให้เข้าผ่าน App แทน | "เปลี่ยนสิทธิ์เป็น Viewer และแชร์ผ่าน App" |
| **Result** | User เห็นเฉพาะข้อมูลแผนกตัวเองตามที่ควรเป็น | "RLS ทำงานถูกต้อง ไม่มีข้อมูลรั่วไหล" |

### 4.2 RCA (Root Cause Analysis) - Deployment Failure
1.  **Why?** ทำไม Report บน Production ถึงแสดงข้อมูล Test?
    → เพราะ Dataset ชี้ไปที่ Database Test
2.  **Why?** ทำไม Dataset ถึงชี้ไปที่ Database Test?
    → เพราะ Data Source Rules ใน Deployment Pipeline ไม่ได้ตั้งค่า
3.  **Why?** ทำไมถึงไม่ได้ตั้งค่า?
    → เพราะขั้นตอนการ Deploy ไม่ได้ระบุให้ตรวจสอบ Rules
4.  **Why?** ทำไมขั้นตอนการ Deploy ถึงไม่มี Checklist?
    → **Root Cause:** ทีมขาด Standard Operating Procedure (SOP) สำหรับการ Deploy
    **Action Item:** สร้าง Checklist บังคับตรวจสอบ Deployment Rules ก่อนกด Deploy ทุกครั้ง

### 4.3 KPI SMART - Advanced Support
| ตัวชี้วัด (KPI) | Specific | Measurable | Achievable | Relevant | Time-bound |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **RLS Accuracy** | อัตราความถูกต้องของสิทธิ์การมองเห็นข้อมูล | 100% | ตรวจสอบด้วย Script อัตโนมัติ | ป้องกัน Data Leakage | วัดผลรายเดือน |
| **Deployment Success Rate** | อัตราความสำเร็จในการ Deploy Report สู่ Prod | ≥ 98% | ใช้ Deployment Pipeline + Checklist | ลด Downtime | วัดผลรายไตรมาส |
| **Dataflow Refresh Success** | อัตราความสำเร็จของ Dataflow | ≥ 99.5% | ใช้ Enhanced Compute Engine | เป็นรากฐานของข้อมูล | วัดผลรายเดือน |

---

## 5. สรุป

### 5.1 ประโยชน์ที่ได้รับ
- **Enterprise-Ready:** คู่มือนี้ครอบคลุมถึงการทำงานในระดับ Enterprise ที่ต้องการความปลอดภัย (RLS) และกระบวนการ Release ที่รัดกุม (Deployment Pipelines)
- **ลดความเสี่ยงทางธุรกิจ:** การป้องกัน Data Leakage และการป้องกันการ Deploy ผิด Environment ช่วยรักษาความน่าเชื่อถือของทีมไอที
- **มาตรฐานเดียวกัน:** ทีม Support มีแนวทางเดียวกันในการรับมือกับปัญหา RLS, Pipeline และ Dataflow

### 5.2 ข้อดี
- การใช้ Deployment Pipeline ช่วยให้การทำงานร่วมกันระหว่าง Developer, Tester และ Support เป็นไปอย่างราบรื่น
- การทำ Dynamic RLS ช่วยลดภาระการจัดการสิทธิ์เมื่อพนักงานมีการเปลี่ยนแปลงตำแหน่ง

### 5.3 ข้อเสีย
- ฟีเจอร์ขั้นสูง (Deployment Pipeline, Dataflow) จำเป็นต้องใช้ Power BI Premium หรือ Fabric Capacity ซึ่งมีค่าใช้จ่าย
- การ Debug ปัญหา RLS และ Pipeline ต้องใช้ความรู้เฉพาะทางและต้องประสานงานกับหลายฝ่าย (Identity Team, Network Team)

### 5.4 ข้อห้าม
- ❌ **ห้ามแก้ไข Report บน Workspace Production โดยตรง** — ต้องผ่าน Pipeline เสมอ
- ❌ **ห้ามลืมตั้งค่า Deployment Rules** — เสี่ยงต่อการเขียนทับข้อมูลจริงด้วยข้อมูล Test
- ❌ **ห้ามให้สิทธิ์ Workspace ระดับ Admin/Member กับ User ที่ต้องใช้ RLS** — สิทธิ์นี้จะทะลุ RLS
- ❌ **ห้ามแก้ไข Dataflow โดยไม่ทำ Impact Analysis** — อาจทำให้ Dataset หลายตัวพังพร้อมกัน

---
**เอกสารอ้างอิง:**
- Microsoft Learn: Row-Level Security, Deployment Pipelines, Dataflows
- EPC Group: Enterprise Power BI Security & Governance
- team400.ai: Power BI Workspace Management & Deployment