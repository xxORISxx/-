<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>日報システム</title>

<style>
body {
    font-family: sans-serif;
    padding: 20px;
}
input, textarea, select, button {
    width: 100%;
    padding: 10px;
    margin-top: 10px;
    font-size: 16px;
}
button {
    cursor: pointer;
}
h2 { margin-top: 30px; }
</style>

</head>
<body>

<h1>日報入力</h1>

<label>① 作業現場名</label>
<input type="text" id="genba">

<label>② 日報作成者（次回以降自動入力）</label>
<input type="text" id="name">

<label>③ 日付</label>
<input type="date" id="date">

<label>④ 作業時間（例：08:00〜11:00）</label>
開始：<input type="time" id="start">
終了：<input type="time" id="end">

<button onclick="calc()">▶ 作業時間を自動計算</button>
<input type="text" id="worktime" readonly placeholder="作業時間（自動計算）">

<label>⑤ 作業内容</label>
<textarea id="naiyo" rows="4"></textarea>

<button onclick="save()">▶ 日報を保存</button>

<h2>📄 保存した日報一覧</h2>
<div id="list"></div>

<h2>⑥ 週1回メール送信用</h2>
<label>送信先メールアドレス</label>
<input type="email" id="mail">

<button onclick="sendMail()">▶ 日報内容をメール送信</button>

<script>
// 初回ロード時に名前を自動入力
document.addEventListener("DOMContentLoaded", () => {
    if(localStorage.name){
        document.getElementById("name").value = localStorage.name
    }
    showList()
})

// 作業時間を計算（休憩時間自動控除）
function calc(){
    const start = document.getElementById("start").value
    const end   = document.getElementById("end").value

    if(!start || !end) return alert("開始と終了時間を入力してください")

    let s = new Date("2020/1/1 " + start)
    let e = new Date("2020/1/1 " + end)
    let min = (e - s) / 60000

    // 休憩時間帯
    const breaks = [
        ["10:00", "10:30"],
        ["12:00", "13:00"],
        ["15:00", "15:30"]
    ]

    breaks.forEach(b => {
        const bs = new Date("2020/1/1 " + b[0])
        const be = new Date("2020/1/1 " + b[1])

        const overlap = Math.min(e, be) - Math.max(s, bs)
        if(overlap > 0){
            min -= overlap / 60000
        }
    })

    if(min < 0) min = 0
    document.getElementById("worktime").value = (min / 60).toFixed(2) + " 時間"
}

// 日報を保存
function save(){
    const name = document.getElementById("name").value
    const data = {
        genba: document.getElementById("genba").value,
        name: name,
        date: document.getElementById("date").value,
        start: document.getElementById("start").value,
        end: document.getElementById("end").value,
        worktime: document.getElementById("worktime").value,
        naiyo: document.getElementById("naiyo").value
    }
    if(!name) return alert("作成者名を入力してください")

    // 名前はローカル記憶
    localStorage.name = name

    // 日報一覧に追加
    let reports = JSON.parse(localStorage.reports || "[]")
    reports.push(data)
    localStorage.reports = JSON.stringify(reports)

    alert("保存しました")
    showList()
}

// 保存リスト表示
function showList(){
    let reports = JSON.parse(localStorage.reports || "[]")
    let html = ""
    reports.forEach((r,i)=>{
        html += `
            <div style="border:1px solid #ccc; padding:10px; margin:10px 0;">
                <b>${r.date}｜${r.genba}</b><br>
                作成者：${r.name}<br>
                作業時間：${r.start}〜${r.end}（${r.worktime}）<br>
                内容：${r.naiyo}
            </div>
        `
    })
    document.getElementById("list").innerHTML = html
}

// メール送信（mailtoを生成）
function sendMail(){
    const email = document.getElementById("mail").value
    if(!email) return alert("メールアドレスを入力してください")

    let reports = JSON.parse(localStorage.reports || "[]")
    if(reports.length === 0) return alert("日報がありません")

    let body = "【週報まとめ】\n\n"
    reports.forEach(r=>{
        body += 
`■ ${r.date}（${r.genba}）
作成者：${r.name}
時間：${r.start}〜${r.end}（${r.worktime}）
内容：${r.naiyo}

`
    })

    const url = `mailto:${email}?subject=週報送付&body=${encodeURIComponent(body)}`
    location.href = url
}
</script>

</body>
</html>