function doGet() {
  return HtmlService.createTemplateFromFile('index')
      .evaluate()
      .setTitle('ระบบจัดการและแสดงข้อมูลตารางทริป (Red Theme)')
      .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL)
      .addMetaTag('viewport', 'width=device-width, initial-scale=1');
}

// ฟังก์ชันดึงรายชื่อชีททั้งหมด เพื่อเอาไปทำ Dropdown เลือกเดือน/ทริป
function getSheetNames() {
  try {
    var ss = SpreadsheetApp.getActiveSpreadsheet();
    var sheets = ss.getSheets();
    return sheets.map(function(sheet) {
      return sheet.getName();
    });
  } catch (e) {
    return ["Sheet1"];
  }
}

// ฟังก์ชันดึงข้อมูลจากชีทที่เลือก
function getSheetData(sheetName) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName(sheetName);
  if (!sheet) {
    sheet = ss.getSheets()[0];
  }
  
  var values = sheet.getDataRange().getValues();
  var data = [];
  var headers = [];
  var startRow = -1;
  
  // ค้นหาแถวที่เป็น Header จริงๆ (แถวที่มีคำว่า GROUP หรือ BAS หรือ JENIS BAS)
  for (var i = 0; i < values.length; i++) {
    for (var j = 0; j < values[i].length; j++) {
      var val = String(values[i][j]).toUpperCase();
      if (val.indexOf('GROUP') > -1 || val.indexOf('BAS') > -1 || val.indexOf('HOTEL') > -1) {
        startRow = i;
        break;
      }
    }
    if (startRow > -1) break;
  }
  
  // ถ้าหาแถว Header ไม่เจอเลย ให้ใช้แถวแรกเป็นหลัก
  if (startRow == -1) startRow = 0;
  
  // จัดการ Header และแปลงค่าว่างให้พร้อมแสดงผล
  headers = values[startRow].map(function(h) { return h ? String(h).trim() : ""; });
  
  // อ่านข้อมูลตั้งแต่แถวถัดจาก Header ลงไป
  for (var r = startRow + 1; r < values.length; r++) {
    var rowData = values[r];
    // ตรวจสอบว่าเป็นแถวที่มีข้อมูลจริงไหม (ไม่ใช่แถวว่างยาวๆ)
    var hasData = rowData.some(function(cell) { return cell !== ""; });
    if (hasData) {
      var obj = {};
      var hasActualContent = false;
      
      headers.forEach(function(header, index) {
        var cellValue = rowData[index] !== undefined ? String(rowData[index]).trim() : "";
        var key = header || "Column_" + (index + 1);
        obj[key] = cellValue;
        if (cellValue) hasActualContent = true;
      });
      
      if (hasActualContent) {
        data.push(obj);
      }
    }
  }
  
  return {
    headers: headers.filter(function(h) { return h !== ""; }), // กรองเอาเฉพาะคอลัมน์ที่มีหัวข้อ
    rows: data
  };
}