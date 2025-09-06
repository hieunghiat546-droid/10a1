<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>10A1</title>
  <meta name="theme-color" content="#ff5c8a" />
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
  <style>
    :root{--accent:#24551d;--bg:#ff7b0086;--card:#c600006a;--muted:#666}
    *{box-sizing:border-box}
    body{margin:0;font-family:Inter, system-ui, -apple-system, 'Segoe UI', Roboto, Arial;background:var(--bg);color:#111}
    header{background:linear-gradient(90deg,#ffd1dc10 0,#670000);padding:14px 18px;display:flex;align-items:center;justify-content:space-between;gap:12px}
    .brand{display:flex;gap:12px;align-items:center}
    .logo-img{width:72px;height:72px;border-radius:12px;object-fit:cover;border:3px solid var(--accent);background:#fff}
    h1{font-size:18px;margin:0}
    nav{display:flex;gap:8px}
    button{background:none;border:0;padding:8px 12px;border-radius:8px;cursor:pointer}
    .container{max-width:1200px;margin:18px auto;padding:0 16px}
    .grid{display:grid;grid-template-columns:1fr;gap:16px}
    @media(min-width:1000px){.grid{grid-template-columns:300px 1fr}}
    .card{background:var(--card);padding:14px;border-radius:12px;box-shadow:0 6px 18px rgba(11,7,21,0.05)}
    .muted{color:var(--muted);font-size:13px}
    .ann-list{display:flex;flex-direction:column;gap:8px}
    .ann{padding:10px;border-radius:8px;background:#fff7fb;border:1px solid #ffe0ec}
    input[type=text],textarea,select{width:100%;padding:8px;border-radius:8px;border:1px solid #e6e6ee;margin-top:6px}
    .small{font-size:13px}
    table{width:100%;border-collapse:collapse}
    th,td{padding:8px;border-bottom:1px solid #f1f1f6;text-align:left}
    .actions{display:flex;gap:8px}
    .btn{background:var(--accent);color:#fff;padding:8px 12px;border-radius:10px;border:0;cursor:pointer;font-weight:600}
    .ghost{background:transparent;border:1px solid #eee;color:#333}
    footer{padding:14px;text-align:center;color:var(--muted);font-size:13px}
    .flex{display:flex;gap:8px;align-items:center}
    .uploader{display:flex;gap:8px;align-items:center}
    .gallery img{width:120px;height:80px;object-fit:cover;border-radius:8px}

    /* Hiệu ứng */
    @keyframes fadeZoom {
      from { opacity:0; transform:scale(0.9); }
      to { opacity:1; transform:scale(1); }
    }
    .fade-zoom { animation: fadeZoom 0.8s ease forwards; }

    @keyframes fadeOut {
      from { opacity:1; }
      to { opacity:0; visibility:hidden; }
    }
    .fade-out { animation: fadeOut 0.5s ease forwards; }
  </style>
</head>
<body>
  <!-- Màn hình đăng nhập -->
  <div id="loginScreen" style="display:flex;flex-direction:column;align-items:center;justify-content:center;height:100vh;background:#f1f1f1;">
    <h2>Đăng nhập trang lớp</h2>
    <input id="username" placeholder="Tài khoản" style="margin:5px;padding:8px;">
    <input id="password" type="password" placeholder="Mật khẩu" style="margin:5px;padding:8px;">
    <button onclick="login()" style="padding:8px 12px;margin-top:10px;">Đăng nhập</button>
    <p id="loginMsg" style="color:red;"></p>
  </div>

  <header>
    <div class="brand">
      <img id="logoImg" src="10a1.jpg" alt="Logo lớp" class="logo-img">
      <div>
        <h1 id="classTitle">Lớp 10A1 — Tri thức dẫn lối</h1>
        <div class="muted">Niên khóa 2025–2026 · Quản trị viên: Cao Thị Huỳnh Hoa</div>
      </div>
    </div>
    <nav>
      <button id="uploadLogoBtn" title="Đổi logo">🖼 Đổi logo</button>
      <button id="toggleDark" title="Bật/tắt chế độ tối">🌙</button>
      <button id="printBtn" title="In trang lớp">🖨 In</button>
      <button id="resetBtn" title="Xóa dữ liệu local">🧹 Reset</button>
      <button onclick="logout()" id="logoutBtn">🚪 Đăng xuất</button>
    </nav>
  </header>

  <main class="container">
    <div class="grid">
      <aside>
        <section class="card">
          <h3>Thông báo nhanh</h3>
          <div class="ann-list" id="annList"></div>
          <hr style="margin:10px 0">
          <input id="annTitle" placeholder="Tiêu đề" />
          <textarea id="annContent" rows="3" placeholder="Nội dung thông báo"></textarea>
          <div style="display:flex;gap:8px;margin-top:8px">
            <button class="btn" id="addAnn">Thêm thông báo</button>
            <button class="ghost" id="clearAnns">Xóa tất cả</button>
          </div>
        </section>

        <section class="card" style="margin-top:12px">
          <h3>Danh sách lớp</h3>
          <div class="muted small">Upload file Excel để load danh sách</div>
          <input type="file" id="excelInput" accept=".xls,.xlsx" />
          <div style="margin-top:12px;max-height:240px;overflow:auto">
            <table id="stuTable"><thead></thead><tbody></tbody></table>
          </div>
          <div style="margin-top:8px"><button class="ghost" id="exportCSV">Xuất CSV</button></div>
        </section>

        <section class="card" style="margin-top:12px">
          <h3>Bảng điểm</h3>
          <div class="muted small">Chọn học sinh để thêm điểm từng môn</div>
          <select id="gradeStudent"><option value="">-- Chọn học sinh --</option></select>
          <div class="flex" style="margin-top:8px"><input id="gradeSubject" placeholder="Môn" /><input id="gradeScore" placeholder="Điểm" style="width:120px" /></div>
          <div style="margin-top:8px"><button class="btn" id="addGrade">Thêm điểm</button></div>
          <div style="margin-top:12px;max-height:160px;overflow:auto"><table id="gradeTable"><thead><tr><th>Học sinh</th><th>Môn</th><th>Điểm</th><th></th></tr></thead><tbody></tbody></table></div>
          <div style="margin-top:8px"><button class="ghost" id="computeRank">Tính trung bình & Xếp hạng</button></div>
          <div style="margin-top:8px"><table id="rankTable"><thead><tr><th>Học sinh</th><th>TB</th><th>Xếp hạng</th></tr></thead><tbody></tbody></table></div>
        </section>
      </aside>

      <section>
        <div class="card">
          <h2>Lịch học</h2>
          <div class="muted small">Nhập ngày/giờ - môn - ghi chú</div>
          <div style="display:flex;gap:8px;margin-top:8px"><input id="schDate" placeholder="YYYY-MM-DD" /><input id="schTime" placeholder="Tiết" /><input id="schTopic" placeholder="Môn / Chủ đề" /><button class="btn" id="addSch">Thêm</button></div>
          <div style="margin-top:12px"><table id="schTable"><thead><tr><th>Ngày</th><th>Tiết</th><th>Chủ đề</th><th></th></tr></thead><tbody></tbody></table></div>
        </div>

        <div class="card" style="margin-top:16px">
          <h2>Bài tập & Deadline</h2>
          <div class="muted small">Ghi bài tập theo môn, đặt hạn nộp</div>
          <div style="display:flex;gap:8px;margin-top:8px"><input id="taskSubject" placeholder="Môn" /><input id="taskTitle" placeholder="Tiêu đề" /><input id="taskDue" placeholder="Hạn (YYYY-MM-DD)" /><button class="btn" id="addTask">Thêm</button></div>
          <div style="margin-top:12px"><table id="taskTable"><thead><tr><th>Môn</th><th>Tiêu đề</th><th>Hạn</th><th></th></tr></thead><tbody></tbody></table></div>
        </div>

        <div class="card" style="margin-top:16px">
          <h2>Thư viện tài liệu</h2>
          <div class="muted small">Thêm link tài liệu hoặc upload file (lưu local)</div>
          <div style="display:flex;gap:8px;margin-top:8px"><input id="docTitle" placeholder="Tiêu đề" /><input id="docUrl" placeholder="Link (hoặc bỏ trống để upload)" /><input id="docFile" type="file" /></div>
          <div style="margin-top:8px"><button class="btn" id="addDoc">Thêm tài liệu</button></div>
          <div style="margin-top:12px"><ul id="docList"></ul></div>
        </div>

        <div class="card" style="margin-top:16px">
          <h2>Gallery kỷ niệm</h2>
          <div class="muted small">Upload ảnh để hiển thị (lưu local)</div>
          <div class="uploader" style="margin-top:8px"><input id="galleryFile" type="file" accept="image/*" multiple /><button class="btn" id="addGallery">Upload</button></div>
          <div class="gallery" id="gallery" style="margin-top:12px;display:flex;flex-wrap:wrap;gap:8px"></div>
        </div>
      </section>
    </div>
  </main>

  <footer>Lớp 10A1 - 2025_2026. Thiết kế bởi Trương Hiếu Nghĩa</footer>
  <input id="logoInput" type="file" accept="image/*" style="display:none">

  <script>
    const $ = id=>document.getElementById(id)

    // --- Login ---
    const ACCOUNTS = [
      {user:"admin", pass:"123456", role:"gv"},
      {user:"hs1", pass:"111111", role:"hs"},
      {user:"hs2", pass:"222222", role:"hs"}
    ]

    function login(){
      let u = $("username").value.trim()
      let p = $("password").value.trim()
      let acc = ACCOUNTS.find(a=>a.user===u && a.pass===p)
      if(acc){
        localStorage.setItem("loggedIn","true")
        localStorage.setItem("role", acc.role)
        $("loginScreen").classList.add("fade-out")
        setTimeout(()=>{
          $("loginScreen").style.display="none"
          let header=document.querySelector("header")
          let main=document.querySelector("main")
          let footer=document.querySelector("footer")
          ;[header,main,footer].forEach(el=>{
            el.style.display="block"
            el.classList.add("fade-zoom")
          })
          applyRole(acc.role)
        },500)
      }else{
        $("loginMsg").textContent="Sai tài khoản hoặc mật khẩu!"
      }
    }

    function applyRole(role){
      if(role==="hs"){
        document.querySelectorAll(".btn, .ghost").forEach(b=>{
          if(b.id!=="logoutBtn") b.style.display="none"
        })
      }
    }

    function logout(){
      localStorage.removeItem("loggedIn")
      localStorage.removeItem("role")
      location.reload()
    }

    window.addEventListener("load",()=>{
      if(localStorage.getItem("loggedIn")==="true"){
        $("loginScreen").style.display="none"
        let role = localStorage.getItem("role")
        applyRole(role)
      }else{
        document.querySelector("header").style.display="none"
        document.querySelector("main").style.display="none"
        document.querySelector("footer").style.display="none"
      }
    })

    // --- Excel Upload ---
    $('excelInput').addEventListener('change', e=>{
      const file = e.target.files[0]
      if(!file) return
      const reader = new FileReader()
      reader.onload = function(evt){
        const data = new Uint8Array(evt.target.result)
        const workbook = XLSX.read(data,{type:'array'})
        const sheetName = workbook.SheetNames[0]
        const worksheet = workbook.Sheets[sheetName]
        const json = XLSX.utils.sheet_to_json(worksheet,{header:1})
        renderExcelTable(json)
        localStorage.setItem('excelStudents', JSON.stringify(json))
      }
      reader.readAsArrayBuffer(file)
    })

    function renderExcelTable(data){
      const thead = document.querySelector('#stuTable thead')
      const tbody = document.querySelector('#stuTable tbody')
      thead.innerHTML = ''
      tbody.innerHTML = ''
      data.forEach((row,i)=>{
        const tr=document.createElement('tr')
        row.forEach(cell=>{
          const td=document.createElement(i===0?'th':'td')
          td.textContent=cell
          tr.appendChild(td)
        })
        if(i===0) thead.appendChild(tr); else tbody.appendChild(tr)
      })
    }

    $('exportCSV').onclick=()=>{
      const data = localStorage.getItem('excelStudents')
      if(!data){ alert('Chưa có dữ liệu'); return }
      const arr = JSON.parse(data)
      let csv = arr.map(r=>r.join(",")).join("\n")
      const blob = new Blob([csv],{type:'text/csv'})
      const url = URL.createObjectURL(blob)
      const a=document.createElement('a')
      a.href=url; a.download='danhsach.csv'; a.click()
    }

    window.addEventListener('load',()=>{
      const data = localStorage.getItem('excelStudents')
      if(data) renderExcelTable(JSON.parse(data))
    })
  </script>
</body>
</html>
