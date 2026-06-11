# Get Data
## File 
## API - HDC
### 📑 ดึงข้อมูลจาก HDC 

Home -> Tranform data -> Home -> New Source  -> Blank Qury
```powerquery
let
    // 1. สร้างรายการปีที่ต้องการดึงข้อมูล (สามารถเพิ่มปีได้)
    YearsList = {"2566", "2567", "2568", "2569"}, 
    
    // 2. แปลงรายการปีให้เป็นตาราง
    YearsTable = Table.FromList(YearsList, Splitter.SplitByNothing(), {"Year"}, null, ExtraValues.Error),
    
    // 3. ฟังก์ชันดึงข้อมูล พร้อมระบบตรวจสอบความผิดพลาด
    GetData = (TargetYear as text) =>
        let
            url = "[https://opendata.moph.go.th/api/report_data](https://opendata.moph.go.th/api/report_data)",
            body = "{
                ""tableName"": ""s_epi1_w"",
                ""year"": """ & TargetYear & """,
                ""province"": ""11"",
                ""type"": ""json""
            }",
            
            // ใช้ try เพื่อดักจับกรณีที่ Web Response เกิด Error
            Response = try Json.Document(Web.Contents(url, [
                Headers = [#"Content-Type"="application/json"],
                Content = Text.ToBinary(body),
                ManualStatusHandling = {400, 404, 500}
            ])),
            
            // เช็คผลลัพธ์: ถ้าเกิด Error หรือได้ผลลัพธ์เป็นตาราง/List ว่างเปล่า ให้คืนค่าเป็น null
            Result = if Response[HasError] then null 
                     else if Response[Value] is list and List.IsEmpty(Response[Value]) then null
                     else if Response[Value] is record and Record.FieldCount(Response[Value]) = 0 then null
                     else Response[Value]
        in
            Result,
            
    // 4. เรียกใช้งานฟังก์ชันเพื่อสร้างคอลัมน์ข้อมูล
    InvokedFunction = Table.AddColumn(YearsTable, "Data", each GetData([Year])),
    
    // 5. กรองเอาแถวที่เป็น null (ปีที่ไม่มีข้อมูลหรือดึงไม่สำเร็จ) ออกไปทันที
    FilteredRows = Table.SelectRows(InvokedFunction, each ([Data] <> null))
in
    FilteredRows

```
### 📑 ดึงข้อมูลจาก HDC - มากกว่า 1 

Home -> Tranform data -> Home -> New Source  -> Blank Qury

