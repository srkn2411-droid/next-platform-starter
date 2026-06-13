// ฟังก์ชันสำหรับส่งข้อมูลให้แอปมือถือผ่านทาง HTTP GET
function doGet(e) {
  var sheetName = e.parameter.sheet; // รับชื่อชีทจากแอปมือถือ เช่น ?sheet=January
  
  if (!sheetName) {
    // ถ้าไม่ได้ระบุชื่อชีท ให้ส่งรายชื่อชีททั้งหมดกลับไปก่อน
    var sheets = getSheetNames();
    return ContentService.createTextOutput(JSON.stringify({ status: "success", type: "sheet_list", data: sheets }))
                         .setMimeType(ContentService.MimeType.JSON);
  }
  
  // ถ้าระบุชื่อชีท ให้ดึงข้อมูลในชีทนั้นส่งกลับไป
  var result = getSheetData(sheetName);
  return ContentService.createTextOutput(JSON.stringify({ status: "success", type: "data", data: result }))
                       .setMimeType(ContentService.MimeType.JSON);
}
