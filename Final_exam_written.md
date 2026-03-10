# Final Exam Written Part Guideline

# Task

คุณทำงานในตำแหน่ง Data Architect ที่กำลังจะนำเสนอ Big Data System ใหม่สำหรับ

- Youtube Trending Analytics System
- Grab Car Platform
- E-Commerce Platform (Shopee/Lazada)
- Smart City Traffic Monitoring for Bangkok
- Platform คนละครึ่งพลัส
- Thailand's Social Security Office (SSO)

หน้าที่ของคุณในฐานะ Data Architect คือการออกแบบ Big Data Architecture โดยใช้ Databricks และ data pipelines ที่สามารถรองรับ data ingestion, integration, processing และ analytics สำหรับระบบ คนละครึ่งพลัส การออกแบบต้องครอบคลุมประเด็นต่อไปนี้:

การออกแบบในแต่ละระบบจะต้องบรรยายประเด็นต่างๆ ดังต่อไปนี้
1. Data Sources ที่จะใช้ มีอะไรบ้าง มาจากไหนบ้าง ใครเป็นเจ้าของ มีรูปแบบข้อมูลอย่างไร (Structured, semi-structured/unstructured)
2. Data Ingestion บรรยายว่าจะนำเข้า data เข้าสู่ระบบได้อย่างไร (เช่น การทำ batch ingestion, การใช้ streaming API, การ upload file และอื่นๆ) รวมถึง tools ที่จะต้องใช้ทั้งในและนอก databricks
3. Data Storage Architecture Design
โครงสร้างของ Data lakehouse (Bronze, Silver, Gold) เก็บข้อมูลอะไรบ้างในแต่ละ Layer
4. Data Pipelines ออกแบบ transformation pipeline ที่เกี่ยวข้องในการ transform, validate และ integrate ข้อมูลในแต่ละ layer รวมถึงการ join dataset ที่เกี่ยวข้อง
5. Data Processing & Analytics
อธิบายว่า data ที่ process มาแล้วสามารถนำมาใช้ทำ analytics ได้อย่างไร และมี use case อะไรบ้างที่จะได้ประโยชน์จาก data นี้
6. System Scalability and Reliability 
อธิบายว่าการออกแบบ architecture และ pipeline ของเราจัดการปริมาณข้อมูลขนาดใหญ่ และการันตี Reliability ได้อย่างไร

# Required Diagrams
1. Big Data System Architecture ที่แสดง components หลักและความสัมพันธ์ (เช่น Data Source, Ingestion Layer, Databricks platform, Storage Layers, Analytics tool และ End Users)
2. Data Pipeline Diagram แสดงถึง flow ของ data ตั้งแต่ต้นทางจนถึง Gold layer พร้อมระบุ transformation ที่เกิดขึ้นในแต่ละขั้นตอน