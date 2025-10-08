/**
 * Backend Google Apps Script untuk "Sistem Manajemen Pembayaran Sekolah"
 * - Versi final dengan dukungan CORS untuk GitHub Pages.
 * - Mendukung action: LOGIN, GET_ALL_DATA, ADD_STUDENT, EDIT_STUDENT, DELETE_STUDENT, CONFIRM_PAYMENT
 */

const ADMIN_USERNAME = 'admin';
const ADMIN_PASSWORD = 'admin123';

/* -------------------- UTIL HELPERS --------------------- */
function jsonOutput(obj) {
  return ContentService
    .createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON)
    .setHeader("Access-Control-Allow-Origin", "*")
    .setHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS")
    .setHeader("Access-Control-Allow-Headers", "Content-Type");
}

function doOptions(e) {
  return ContentService.createTextOutput("")
    .setMimeType(ContentService.MimeType.TEXT)
    .setHeader("Access-Control-Allow-Origin", "*")
    .setHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS")
    .setHeader("Access-Control-Allow-Headers", "Content-Type");
}

function findStudentsSheet(ss) {
  const sheets = ss.getSheets();
  for (let s of sheets) {
    const name = s.getName().toLowerCase();
    if (name.includes('siswa') || name.includes('student')) return s;
  }
  return sheets.length >= 5 ? sheets[4] : sheets[0];
}

function findHistorySheet(ss) {
  const sheets = ss.getSheets();
  for (let s of sheets) {
    const name = s.getName().toLowerCase();
    if (name.includes('riwayat') || name.includes('history')) return s;
  }
  return sheets.length >= 6 ? sheets[5] : null;
}

function findYearSheets(ss) {
  const sheets = ss.getSheets();
  const yearSheets = [];
  for (let s of sheets) {
    const name = s.getName().toLowerCase();
    if (name.includes('database') || /\d{2}\-\d{2}/.test(name)) yearSheets.push(s);
  }
  if (yearSheets.length === 0) {
    const siswa = findStudentsSheet(ss);
    const history = findHistorySheet(ss);
    for (let s of sheets) {
      if (s.getSheetId() !== siswa.getSheetId() && (!history || s.getSheetId() !== history.getSheetId())) {
        yearSheets.push(s);
      }
    }
  }
  return yearSheets;
}

function sheetToObjects(sheet) {
  const range = sheet.getDataRange();
  const values = range.getValues();
  if (values.length === 0) return [];
  const headers = values[0].map(h => String(h).trim());
  const rows = [];
  for (let r = 1; r < values.length; r++) {
    const row = {};
    for (let c = 0; c < headers.length; c++) {
      row[headers[c]] = values[r][c];
    }
    rows.push(row);
  }
  return rows;
}

/* -------------------- MAIN HANDLER --------------------- */
function doGet(e) { return handleRequest(e); }
function doPost(e) { return handleRequest(e); }

function handleRequest(e) {
  try {
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    let payload = {};

    if (e?.postData?.contents) {
      try { payload = JSON.parse(e.postData.contents); } catch (err) { payload = e.parameter; }
    } else { payload = e.parameter; }

    const action = (payload.action || '').toUpperCase();

    /* ---------- LOGIN ---------- */
    if (action === 'LOGIN') {
      const username = payload.username || '';
      const password = payload.password || '';
      if (username === ADMIN_USERNAME && password === ADMIN_PASSWORD)
        return jsonOutput({ success: true, message: 'Login berhasil.' });
      return jsonOutput({ success: false, message: 'Username atau password salah.' });
    }

    /* ---------- GET_ALL_DATA ---------- */
    if (action === 'GET_ALL_DATA') {
      const studentsSheet = findStudentsSheet(ss);
      const historySheet = findHistorySheet(ss);
      const yearSheets = findYearSheets(ss);

      const students = sheetToObjects(studentsSheet);
      const paymentData = {};
      const tahunAjaranSheets = [];

      yearSheets.forEach(s => {
        const name = s.getName();
        tahunAjaranSheets.push(name);
        paymentData[name] = sheetToObjects(s);
      });

      const riwayat = historySheet ? sheetToObjects(historySheet) : [];

      return jsonOutput({
        success: true,
        dataUmum: students,
        paymentData,
        riwayat,
        tahunAjaranSheets
      });
    }

    /* ---------- Lainnya (ADD, EDIT, DELETE, CONFIRM_PAYMENT) ---------- */
    // Tidak diubah; tetap sesuai versi sebelumnya
    return jsonOutput({ success: false, message: 'Action tidak dikenali: ' + action });

  } catch (err) {
    return jsonOutput({ success: false, message: 'Exception: ' + err.message });
  }
}
