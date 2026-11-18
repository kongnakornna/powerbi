---

# คู่มือการทำงาน Application Support: Power BI

**ผู้จัดทำ:** ทีม Application Support
**เวอร์ชัน:** 1.0
**สถานะ:** ต้นฉบับ

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