<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>せんせいアシスト(教育相談編) - 自立活動 状態把握 v0.18</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
<style>
  * { box-sizing: border-box; }
  body {
    font-family: -apple-system, "Hiragino Kaku Gothic ProN", "Yu Gothic", sans-serif;
    background: #0a0b0f;
    background-image: radial-gradient(circle at 20% -10%, rgba(124,92,255,0.12), transparent 45%),
                       radial-gradient(circle at 90% 10%, rgba(34,211,238,0.08), transparent 40%);
    color: #eceef5;
    margin: 0;
    padding: 16px;
  }
  .wrap { max-width: 980px; margin: 0 auto; }
  h1 { font-size: 21px; font-weight: 700; margin: 0 0 4px; color: #fff; letter-spacing: 0.2px; }
  .subtitle { font-size: 13px; color: #7d8096; margin-bottom: 20px; }

  .card {
    background: linear-gradient(180deg, #14151d 0%, #101116 100%);
    border: 1px solid #23242f;
    border-radius: 14px;
    padding: 16px 20px;
    margin-bottom: 16px;
    box-shadow: 0 1px 0 rgba(255,255,255,0.03) inset, 0 8px 24px rgba(0,0,0,0.25);
  }
  .card h2 {
    font-size: 13px; font-weight: 700; margin: 0 0 12px; color: #9d9fe8;
    text-transform: uppercase; letter-spacing: 0.08em;
  }

  .basic-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
    gap: 12px;
  }
  .basic-grid label {
    font-size: 12px;
    color: #7d8096;
    display: block;
    margin-bottom: 4px;
  }
  .basic-grid input, .basic-grid select {
    width: 100%;
    padding: 8px;
    font-size: 14px;
    border: 1px solid #2a2c3a;
    border-radius: 8px;
    background: #0e0f14;
    color: #eceef5;
    color-scheme: dark;
  }
  .basic-grid input:focus, .basic-grid select:focus {
    outline: none;
    border-color: #7c5cff;
    box-shadow: 0 0 0 3px rgba(124,92,255,0.2);
  }

  .chip-field { margin-top: 14px; }
  .chip-field label { font-size: 12px; color: #7d8096; display: block; margin-bottom: 6px; }
  .chips-multi { display: flex; flex-wrap: wrap; gap: 6px; }
  .chip {
    font-size: 12px;
    padding: 5px 12px;
    border-radius: 14px;
    border: 1px solid #2a2c3a;
    background: #0e0f14;
    cursor: pointer;
    color: #a7a9bd;
    transition: all 0.15s ease;
  }
  .chip.selected {
    background: linear-gradient(135deg, #7c5cff, #22d3ee);
    color: #0a0b0f;
    border-color: transparent;
    font-weight: 700;
  }

  .main-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 16px;
  }
  @media (max-width: 720px) {
    .main-grid { grid-template-columns: 1fr; }
  }

  #tabs { display: flex; flex-wrap: wrap; gap: 6px; margin-bottom: 14px; }
  #tabs button {
    font-size: 13px;
    padding: 6px 14px;
    border: 1px solid #2a2c3a;
    border-radius: 20px;
    background: #0e0f14;
    cursor: pointer;
    color: #a7a9bd;
    transition: all 0.15s ease;
  }
  #tabs button.active {
    background: linear-gradient(135deg, #7c5cff, #5b8dff);
    color: #fff;
    border-color: transparent;
    box-shadow: 0 0 0 1px rgba(124,92,255,0.4), 0 4px 14px rgba(124,92,255,0.35);
  }

  .item-row { margin-bottom: 18px; }
  .item-row .item-head {
    display: flex;
    justify-content: space-between;
    font-size: 13px;
    color: #d5d7e6;
    margin-bottom: 4px;
  }
  .item-row .item-head .val { font-weight: 700; color: #9d9fe8; min-width: 60px; text-align: right; }

  input[type=range] {
    width: 100%;
    accent-color: #7c5cff;
    background: transparent;
  }
  .slider-anchors {
    display: flex;
    justify-content: space-between;
    font-size: 11px;
    color: #5c5e70;
    margin-top: 2px;
  }

  .detail-toggle {
    font-size: 12px;
    color: #22d3ee;
    background: none;
    border: none;
    cursor: pointer;
    padding: 0;
    margin-top: 6px;
    font-weight: 600;
  }
  .detail-box {
    display: none;
    margin-top: 8px;
    padding: 10px;
    background: #0e0f14;
    border: 1px solid #23242f;
    border-radius: 10px;
  }
  .detail-box.open { display: block; }
  .detail-box .chips { display: flex; flex-wrap: wrap; gap: 6px; margin-bottom: 8px; }
  .detail-box textarea {
    width: 100%;
    font-size: 13px;
    padding: 8px;
    border: 1px solid #2a2c3a;
    border-radius: 8px;
    resize: vertical;
    min-height: 50px;
    font-family: inherit;
    background: #14151d;
    color: #eceef5;
  }

  .radar-pane { position: sticky; top: 16px; }
  #radarNote {
    font-size: 12px;
    color: #7d8096;
    margin-top: 8px;
    line-height: 1.6;
  }

  .save-bar { text-align: right; margin-top: 8px; }
  .save-bar button {
    font-size: 13px;
    font-weight: 700;
    padding: 9px 20px;
    border-radius: 10px;
    border: none;
    background: linear-gradient(135deg, #7c5cff, #22d3ee);
    color: #0a0b0f;
    cursor: pointer;
    box-shadow: 0 4px 16px rgba(124,92,255,0.3);
  }
  .status-msg { font-size: 12px; color: #22d3ee; margin-right: 10px; }
  button.chip { font-family: inherit; }
  .chip:focus-visible, button:focus-visible, input:focus-visible, select:focus-visible, textarea:focus-visible { outline: 3px solid #22d3ee; outline-offset: 2px; }
  .notice { border-color: #22d3ee; }
  .notice h2 { color: #22d3ee; text-transform: none; letter-spacing: 0.02em; font-size: 15px; }
  .notice p { font-size: 14px; line-height: 1.8; margin: 0 0 6px; color: #d5d7e6; }
  .notice-more { margin-top: 8px; font-size: 13.5px; color: #d5d7e6; }
  .notice-more summary { cursor: pointer; color: #22d3ee; font-weight: 700; padding: 4px 0; }
  .notice-more ul { margin: 8px 0 0; padding-left: 20px; line-height: 1.8; }
  .ghost-btn { font-size: 13px; font-weight: 700; padding: 8px 16px; border-radius: 10px; border: 1px solid #22d3ee; background: transparent; color: #22d3ee; cursor: pointer; font-family: inherit; }
</style>
</head>
<body>
<div class="wrap">
  <h1>せんせいアシスト <span style="font-weight:400; color:#7d8096; font-size:15px;">教育相談編 ― 自立活動 状態把握(v0.18)</span></h1>
  <div class="subtitle">27項目をざっくり動かして、レーダーチャートで全体像を見る</div>

  <div class="card notice" role="note" aria-labelledby="notice-title">
    <h2 id="notice-title">はじめに：架空の児童生徒で、典型的な支援を想定する道具です</h2>
    <p>このアプリは、<strong>架空の児童生徒</strong>の特性（学年、障害種別、自立活動の各項目の様子など）を入れ、<strong>そのような特性をもつ児童生徒に典型的に考えられる支援</strong>を想定するためのものです。研修や校内での話し合い、支援の引き出しを増やす練習に使えます。</p>
    <p><strong>実際の児童生徒のデータは入力しないでください。</strong>氏名や記号に置きかえても、実在の児童生徒の様子をもとにした入力はしないでください。入力した内容は、AI（Gemini）に送られます。</p>
    <p>出てくるのは、架空の特性から想定した典型的な支援の例です。特定の児童生徒の見立てや指導方針として使うものではなく、診断や医学的な判断を行うものでもありません。AIの出力には誤りが含まれることがあります。</p>
    <details class="notice-more">
      <summary>入力した内容はどこへ行くか・使うときの決まり</summary>
      <ul>
        <li><strong>運営者（toi toi toi）は、入力内容を受け取りません。</strong>入力は、使う人のブラウザから、その人が自分でデプロイした Google Apps Script を通して、Google の Gemini API に送られます。</li>
        <li><strong>Gemini API の無料枠では、送った内容が Google の製品改善に使われ、人が読むことがあります</strong>（Google の Gemini API 追加利用規約）。有料枠ではこの利用はされません。どちらの場合も、実際の児童生徒のデータは入れないでください。</li>
        <li>Gemini API は18歳未満は使えません。<strong>教職員など大人が使う道具</strong>です。児童生徒には使わせないでください。</li>
        <li>学校で使うときは、所属する教育委員会の情報セキュリティポリシーや、生成AIの利用に関する決まりに従ってください。</li>
        <li>このブラウザには、GAS の URL と合言葉だけを保存します（ほかの入力は保存しません）。「JSONで保存」「Googleドキュメントにする」を押したときだけ、結果が自分のパソコンやドライブに残ります。</li>
      </ul>
    </details>
    <div style="text-align:right; margin-top:8px;"><button type="button" class="ghost-btn" onclick="loadSample()">架空の例を入れてみる</button></div>
  </div>

  <div class="card">
    <h2>基本情報</h2>
    <div class="basic-grid">
      <div>
        <label for="caseId">架空の児童生徒の呼び名</label>
        <input type="text" id="caseId" placeholder="例: 架空のAさん">
      </div>
      <div>
        <label for="grade">学年</label>
        <select id="grade">
          <option>小学部低学年</option>
          <option>小学部中学年</option>
          <option>小学部高学年</option>
          <option>中学部</option>
          <option selected>高等部</option>
        </select>
      </div>
      <div>
        <label for="placement">在籍形態</label>
        <select id="placement">
          <option>通常学級</option>
          <option selected>通級</option>
          <option>特別支援学級</option>
          <option>特別支援学校</option>
          <option>訪問教育</option>
        </select>
      </div>
      <div style="position:relative;">
        <label for="visitDate">作成日</label>
        <input type="text" id="visitDate" readonly placeholder="日付を選択" style="width:100%; padding:8px; font-size:14px; border:1px solid #2a2c3a; border-radius:8px; background:#0e0f14; color:#eceef5; cursor:pointer;">
        <div id="calendarPopup" style="display:none; position:absolute; z-index:20; background:#14151d; border:1px solid #23242f; border-radius:10px; padding:10px; margin-top:4px; width:230px; box-shadow:0 8px 24px rgba(0,0,0,0.4);">
          <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:8px;">
            <button type="button" id="calPrev" style="background:none;border:none;color:#a7a9bd;cursor:pointer;font-size:16px;">‹</button>
            <span id="calLabel" style="font-size:13px; color:#eceef5; font-weight:600;"></span>
            <button type="button" id="calNext" style="background:none;border:none;color:#a7a9bd;cursor:pointer;font-size:16px;">›</button>
          </div>
          <div id="calGrid" style="display:grid; grid-template-columns: repeat(7,1fr); gap:4px; font-size:12px; text-align:center;"></div>
        </div>
      </div>
    </div>
    <div class="chip-field">
      <label id="dis-label">架空の児童生徒の障害種別(複数選択・重複障害対応)</label>
      <div class="chips-multi" id="disabilityChips" role="group" aria-labelledby="dis-label"></div>
      <div style="margin-top:10px;">
        <label for="diagnosisText">想定する診断名・病名(任意)</label>
        <input type="text" id="diagnosisText" placeholder="例: 脳室周囲白質軟化症による脳性麻痺" style="width:100%; padding:8px; font-size:14px; border:1px solid #2a2c3a; border-radius:8px; background:#0e0f14; color:#eceef5;">
      </div>
    </div>
  </div>

  <div class="card">
    <h2>自立活動 6区分27項目</h2>
    <div class="chip-field" style="margin-top:0; margin-bottom:16px; padding:12px; background:#0e0f14; border:1px solid #2a2c3a; border-radius:10px;">
      <label for="observationText">架空の児童生徒の様子(文章)から27項目を推定(任意・時短用)</label>
      <textarea id="observationText" placeholder="例: 移動教室はいつも一番後ろ。挨拶や受け答えははっきりしていて先生とはよく話すが、友達とは1〜2人としか関わらない。学習は前向きだが早く終わらせたがる。" style="width:100%; font-size:13px; padding:8px; border:1px solid #2a2c3a; border-radius:8px; background:#14151d; color:#eceef5; resize:vertical; min-height:60px; font-family:inherit;"></textarea>
      <div style="text-align:right; margin-top:8px;">
        <span class="status-msg" id="estimateStatus"></span>
        <button onclick="estimateRadar()" style="font-size:12px; font-weight:700; padding:7px 16px; border-radius:8px; border:none; background:linear-gradient(135deg,#7c5cff,#22d3ee); color:#0a0b0f; cursor:pointer;">この内容から27項目を推定する</button>
      </div>
      <div style="font-size:11px; color:#5c5e70; margin-top:6px;">推定はあくまで叩き台です。下のスライダーで確認・修正してください。</div>
    </div>
    <div class="main-grid">
      <div>
        <div id="tabs"></div>
        <div id="sliderPane"></div>
      </div>
      <div class="radar-pane">
        <div style="position:relative; height:320px;">
          <canvas id="radar" role="img" aria-label="6区分の平均値を示すレーダーチャート"></canvas>
        </div>
        <div id="radarNote">
          スライダーはまず「ざっくりの感触」でOK。左が「困難が大きい」、右が「困難が少ない・できている」を表します。レーダーチャートを見て気になる項目があれば、下の「詳細を入力」を開いて補足してください。
        </div>
      </div>
    </div>
    <div class="save-bar">
      <span class="status-msg" id="statusMsg"></span>
      <button onclick="saveJson()">この内容をJSONで保存</button>
    </div>
  </div>

  <div class="card">
    <h2>支援者(指導する先生)の情報</h2>
    <div class="chip-field" style="margin-top:0;">
      <label id="teacher-label">得意な指導スタイル(複数選択)</label>
      <div class="chips-multi" id="teacherChips" role="group" aria-labelledby="teacher-label"></div>
      <div style="margin-top:10px;">
        <label for="teacherNote">その他・自由記述(任意)</label>
        <textarea id="teacherNote" placeholder="例: 工作や自助具の工夫が得意。じっくり関わる時間は取りにくい。" style="width:100%; font-size:13px; padding:8px; border:1px solid #2a2c3a; border-radius:8px; background:#0e0f14; color:#eceef5; resize:vertical; min-height:44px; font-family:inherit;"></textarea>
      </div>
    </div>
  </div>

  <div class="card">
    <h2>AIによる指導方針の提案</h2>
    <div class="basic-grid" style="margin-bottom:10px;">
      <div style="grid-column: 1 / -1;">
        <label for="gasUrl">GAS WebアプリのURL(自分でデプロイしたURLを貼り付け。このブラウザにだけ保存されます)</label>
        <input type="text" id="gasUrl" placeholder="https://script.google.com/macros/s/.../exec">
      </div>
      <div style="grid-column: 1 / -1;">
        <label for="appKey">合言葉(GAS の APP_KEY に設定したもの。このブラウザにだけ保存されます)</label>
        <input type="password" id="appKey" autocomplete="off">
      </div>
    </div>
    <div class="save-bar" style="margin-top:0;">
      <span class="status-msg" id="aiStatus"></span>
      <button onclick="askAi()">この内容から指導方針を提案してもらう</button>
    </div>
    <div id="aiOutput" aria-live="polite"></div>
  </div>
</div>

<script>
// --- GAS WebアプリのURL ---
// ここに自分のURL(https://script.google.com/macros/s/.../exec の形)を書き込んでおくと、
// 開くたびに毎回貼り直さなくてよくなります。空欄のままなら、今まで通り手入力してください。
// 公開版では空欄にしています。自分のURLをここに書くと毎回貼らずに済みますが、
// そのファイルを他の人に渡すと、あなたのAPI利用枠が使われてしまうので注意してください。
const DEFAULT_GAS_URL = '';
(function () {
  const el = document.getElementById('gasUrl');
  let saved = '';
  try { saved = localStorage.getItem('senseiAssistGasUrl') || ''; } catch (e) {}
  el.value = DEFAULT_GAS_URL || saved;
  el.addEventListener('change', () => { try { localStorage.setItem('senseiAssistGasUrl', el.value.trim()); } catch (e) {} });
  const k = document.getElementById('appKey');
  try { k.value = localStorage.getItem('senseiAssistAppKey') || ''; } catch (e) {}
  k.addEventListener('change', () => { try { localStorage.setItem('senseiAssistAppKey', k.value); } catch (e) {} });
})();
// GAS に送るときは、合言葉を添える
function withKey(obj) {
  return JSON.stringify(Object.assign({}, obj, { appKey: document.getElementById('appKey').value }));
}

// --- 月曜始まりカレンダー ---
const visitDateInput = document.getElementById('visitDate');
const calendarPopup = document.getElementById('calendarPopup');
const calLabel = document.getElementById('calLabel');
const calGrid = document.getElementById('calGrid');
let calYear, calMonth; // calMonth: 0-11
const today = new Date();
calYear = today.getFullYear();
calMonth = today.getMonth();

function renderCalendar() {
  calLabel.textContent = `${calYear}年 ${calMonth + 1}月`;
  calGrid.innerHTML = '';
  const dowLabels = ['月','火','水','木','金','土','日'];
  dowLabels.forEach(d => {
    const cell = document.createElement('div');
    cell.textContent = d;
    cell.style.color = '#7d8096';
    cell.style.padding = '4px 0';
    calGrid.appendChild(cell);
  });
  const firstDay = new Date(calYear, calMonth, 1);
  const offset = (firstDay.getDay() + 6) % 7; // 月曜=0になるよう調整
  const daysInMonth = new Date(calYear, calMonth + 1, 0).getDate();
  for (let i = 0; i < offset; i++) {
    calGrid.appendChild(document.createElement('div'));
  }
  for (let day = 1; day <= daysInMonth; day++) {
    const cell = document.createElement('button');
    cell.type = 'button';
    cell.textContent = day;
    const isToday = (calYear === today.getFullYear() && calMonth === today.getMonth() && day === today.getDate());
    cell.style.cssText = 'background:none;cursor:pointer;padding:5px 0;border-radius:6px;'
      + (isToday
          ? 'border:1px solid #22d3ee; color:#22d3ee; font-weight:700;'
          : 'border:none; color:#eceef5;');
    cell.onmouseenter = () => { if (!isToday) cell.style.background = '#23242f'; };
    cell.onmouseleave = () => { if (!isToday) cell.style.background = 'none'; };
    cell.onclick = () => {
      const mm = String(calMonth + 1).padStart(2, '0');
      const dd = String(day).padStart(2, '0');
      visitDateInput.value = `${calYear}-${mm}-${dd}`;
      calendarPopup.style.display = 'none';
    };
    calGrid.appendChild(cell);
  }
}

visitDateInput.addEventListener('click', (e) => {
  e.stopPropagation();
  calendarPopup.style.display = calendarPopup.style.display === 'none' ? 'block' : 'none';
  renderCalendar();
});
document.getElementById('calPrev').addEventListener('click', (e) => {
  e.stopPropagation();
  calMonth--; if (calMonth < 0) { calMonth = 11; calYear--; }
  renderCalendar();
});
document.getElementById('calNext').addEventListener('click', (e) => {
  e.stopPropagation();
  calMonth++; if (calMonth > 11) { calMonth = 0; calYear++; }
  renderCalendar();
});
document.addEventListener('click', (e) => {
  if (!calendarPopup.contains(e.target) && e.target !== visitDateInput) {
    calendarPopup.style.display = 'none';
  }
});

const teacherStyleOptions = [
  "体を動かすこと","視覚的な教材づくり","ICT活用","言葉かけ・対話",
  "観察と記録","工作・自助具の工夫","音楽・リズムを使った指導"
];
const selectedTeacherStyles = new Set();
const teacherWrap = document.getElementById('teacherChips');
teacherStyleOptions.forEach(name => {
  const c = document.createElement('button');
  c.type = 'button';
  c.className = 'chip';
  c.textContent = name;
  c.setAttribute('aria-pressed', 'false');
  c.onclick = () => {
    if (selectedTeacherStyles.has(name)) { selectedTeacherStyles.delete(name); c.classList.remove('selected'); }
    else { selectedTeacherStyles.add(name); c.classList.add('selected'); }
    c.setAttribute('aria-pressed', String(selectedTeacherStyles.has(name)));
  };
  teacherWrap.appendChild(c);
});

const disabilityOptions = [
  "視覚障害","聴覚障害","知的障害","肢体不自由","病弱・身体虚弱","言語障害",
  "自閉スペクトラム症","ADHD","学習障害(LD)","情緒障害","重複障害","診断なし(教育的ニーズのみ)"
];
const selectedDisabilities = new Set();
const disWrap = document.getElementById('disabilityChips');
disabilityOptions.forEach(name => {
  const c = document.createElement('button');
  c.type = 'button';
  c.className = 'chip';
  c.textContent = name;
  c.setAttribute('aria-pressed', 'false');
  c.onclick = () => {
    if (selectedDisabilities.has(name)) { selectedDisabilities.delete(name); c.classList.remove('selected'); }
    else { selectedDisabilities.add(name); c.classList.add('selected'); }
    c.setAttribute('aria-pressed', String(selectedDisabilities.has(name)));
  };
  disWrap.appendChild(c);
});

const cats = [
  { name: "健康の保持", items: [
    { label: "生活のリズムや生活習慣の形成", chips: ["睡眠時間が短い","朝起きられない","夜型","服薬に伴うリズム変動","週末と平日の差が大きい"] },
    { label: "病気の状態の理解と生活管理", chips: ["体調不良を自分から言えない","薬の意味を理解していない","発作等の予兆に気づけない","無理をしてしまう傾向"] },
    { label: "身体各部の状態の理解と養護", chips: ["姿勢の崩れに気づきにくい","怪我・痛みに気づきにくい","装具や補助具への意識が薄い"] },
    { label: "障害の特性の理解と生活環境の調整", chips: ["自分の特性を人に説明できない","配慮を求める言葉を持たない","配慮を受けることに抵抗がある"] },
    { label: "健康状態の維持・改善", chips: ["運動習慣","食事管理","休息の取り方","通院・リハビリへの主体的な参加"] }
  ]},
  { name: "心理的な安定", items: [
    { label: "情緒の安定", chips: ["不安が強い","失敗への反応が大きい","評価場面で緊張する","パニックになりやすい"] },
    { label: "状況の理解と変化への対応", chips: ["予定変更に弱い","見通しが必要","切替えに時間がかかる"] },
    { label: "困難を改善しようとする意欲", chips: ["諦めやすい","失敗経験の蓄積","自己評価が低い","成功体験が少ない"] }
  ]},
  { name: "人間関係の形成", items: [
    { label: "他者とのかかわりの基礎", chips: ["人への関心が薄い","安心できる相手が限られる","大人には話せるが友達には話せない"] },
    { label: "他者の意図や感情の理解", chips: ["表情理解が難しい","冗談・皮肉が伝わりにくい","相手の意図を誤解しやすい"] },
    { label: "自己の理解と行動の調整", chips: ["距離感がつかみにくい","順番や共有が苦手","自分の行動を振り返るのが苦手"] },
    { label: "集団への参加の基礎", chips: ["集団に入りづらい","孤立しがち","トラブル時の対応が苦手"] }
  ]},
  { name: "環境の把握", items: [
    { label: "保有する感覚の活用", chips: ["視覚優位","聴覚優位","感覚の活用にムラがある"] },
    { label: "感覚や認知の特性への対応", chips: ["感覚過敏","感覚鈍麻","感覚探求行動"] },
    { label: "感覚の補助及び代行手段の活用", chips: ["拡大教材が必要","音声教材が必要","補聴・保有視覚の活用"] },
    { label: "状況把握と行動", chips: ["複数情報の同時処理が苦手","雑音下での情報把握が苦手","人混みが苦手"] },
    { label: "認知や行動の手掛かりとなる概念の形成", chips: ["空間把握が苦手","時間概念が弱い","因果関係の理解が弱い"] }
  ]},
  { name: "身体の動き", items: [
    { label: "姿勢と運動・動作の基本的技能", chips: ["姿勢保持が難しい","筋緊張の偏り","バランスが取りにくい"] },
    { label: "姿勢保持と運動・動作の補助的手段の活用", chips: ["装具の使用","座位保持装置","歩行補助具"] },
    { label: "日常生活に必要な基本動作", chips: ["更衣に時間がかかる","食事動作に介助が必要","排泄動作に時間がかかる"] },
    { label: "身体の移動能力", chips: ["階段昇降に介助が必要","長距離移動が難しい","車椅子操作"] },
    { label: "作業に必要な動作と円滑な遂行", chips: ["書字動作に疲労が大きい","手先の操作が苦手","両手動作の協調が苦手"] }
  ]},
  { name: "コミュニケーション", items: [
    { label: "コミュニケーションの基礎的能力", chips: ["会話の開始が苦手","援助要請ができない","Yes/Noが不正確"] },
    { label: "言語の受容と表出", chips: ["長い説明の理解が難しい","抽象語理解が弱い","語彙が少ない"] },
    { label: "言語の形成と活用", chips: ["文法的な誤りが多い","語想起が難しい","会話の維持が苦手"] },
    { label: "コミュニケーション手段の選択と活用", chips: ["音声以外の手段が必要","AAC・機器の活用","筆談・文字の活用"] },
    { label: "状況に応じたコミュニケーション", chips: ["相手や場面で対応が変わる","電話・オンラインが苦手","初対面で固まる"] }
  ]}
];

const state = cats.map(c => c.items.map(() => ({ val: 50, chips: [], note: "" })));
let active = 0;

const tabsEl = document.getElementById('tabs');
cats.forEach((c, i) => {
  const b = document.createElement('button');
  b.textContent = c.name;
  b.type = 'button';
  b.onclick = () => { active = i; render(); };
  b.id = 'tab-' + i;
  tabsEl.appendChild(b);
});

function styleTabs() {
  cats.forEach((c, i) => {
    const el = document.getElementById('tab-' + i);
    el.className = (i === active) ? 'active' : '';
    el.setAttribute('aria-pressed', String(i === active));
  });
}

function render() {
  styleTabs();
  const pane = document.getElementById('sliderPane');
  pane.innerHTML = '';
  cats[active].items.forEach((item, idx) => {
    const s = state[active][idx];
    const row = document.createElement('div');
    row.className = 'item-row';
    row.innerHTML = `
      <div class="item-head">
        <span>${item.label}</span>
        <span class="val" id="val-${active}-${idx}">${s.val}</span>
      </div>
      <input type="range" min="0" max="100" step="1" value="${s.val}" id="s-${active}-${idx}" aria-label="${item.label}(0=困難が大きい、100=できている)">
      <div class="slider-anchors"><span>困難が大きい</span><span>できている・困難が少ない</span></div>
      <button type="button" class="detail-toggle" id="toggle-${active}-${idx}" aria-expanded="false" aria-controls="detail-${active}-${idx}">詳細を入力 ▾</button>
      <div class="detail-box" id="detail-${active}-${idx}">
        <div class="chips">
          ${item.chips.map((c, ci) => `<button type="button" class="chip" data-ci="${ci}" aria-pressed="false">${c}</button>`).join('')}
        </div>
        <textarea aria-label="${item.label}の補足" placeholder="想定する行動の例があれば記入(例: 3回促しても着手しない)">${s.note}</textarea>
      </div>
    `;
    pane.appendChild(row);

    row.querySelector('input[type=range]').addEventListener('input', e => {
      s.val = Number(e.target.value);
      document.getElementById(`val-${active}-${idx}`).textContent = s.val;
      updateChart();
    });

    row.querySelector('.detail-toggle').addEventListener('click', () => {
      const box = document.getElementById(`detail-${active}-${idx}`);
      box.classList.toggle('open');
      row.querySelector('.detail-toggle').setAttribute('aria-expanded', String(box.classList.contains('open')));
    });

    row.querySelectorAll('.chip').forEach(chip => {
      const ci = Number(chip.dataset.ci);
      if (s.chips.includes(ci)) { chip.classList.add('selected'); chip.setAttribute('aria-pressed', 'true'); }
      chip.addEventListener('click', () => {
        if (s.chips.includes(ci)) {
          s.chips = s.chips.filter(x => x !== ci);
          chip.classList.remove('selected');
        } else {
          s.chips.push(ci);
          chip.classList.add('selected');
        }
        chip.setAttribute('aria-pressed', String(s.chips.includes(ci)));
      });
    });

    row.querySelector('textarea').addEventListener('input', e => {
      s.note = e.target.value;
    });
  });
}

function averages() {
  return state.map(arr => Math.round(arr.reduce((a, b) => a + b.val, 0) / arr.length));
}

let chart;
function updateChart() {
  chart.data.datasets[0].data = averages();
  chart.update();
}

render();

chart = new Chart(document.getElementById('radar'), {
  type: 'radar',
  data: {
    labels: cats.map(c => c.name),
    datasets: [{
      label: '現在の全体感',
      data: averages(),
      backgroundColor: 'rgba(124,92,255,0.22)',
      borderColor: '#8b7bff',
      pointBackgroundColor: '#22d3ee',
      pointBorderColor: '#22d3ee',
      borderWidth: 2
    }]
  },
  options: {
    responsive: true,
    maintainAspectRatio: false,
    scales: {
      r: {
        min: 0, max: 100,
        ticks: { display: false, backdropColor: 'transparent' },
        pointLabels: { font: { size: 12 }, color: '#c7c9db' },
        grid: { color: '#23242f' },
        angleLines: { color: '#23242f' }
      }
    },
    plugins: { legend: { display: false } }
  }
});

async function estimateRadar() {
  const gasUrl = document.getElementById('gasUrl').value.trim();
  const text = document.getElementById('observationText').value.trim();
  const statusEl = document.getElementById('estimateStatus');
  if (!gasUrl) {
    statusEl.textContent = 'GASのURLを入力してください';
    statusEl.style.color = '#f87171';
    return;
  }
  if (!text) {
    statusEl.textContent = '観察の言葉を入力してください';
    statusEl.style.color = '#f87171';
    return;
  }
  statusEl.style.color = '#22d3ee';
  statusEl.textContent = '推定中です…';
  try {
    const itemLabels = [];
    cats.forEach(c => c.items.forEach(it => itemLabels.push(`${c.name}:${it.label}`)));
    const res = await fetch(gasUrl, {
      method: 'POST',
      body: withKey({ action: 'estimate_radar', text: text, itemLabels: itemLabels })
    });
    const json = await res.json();
    if (json.ok && Array.isArray(json.values)) {
      let idx = 0;
      cats.forEach((c, ci) => c.items.forEach((it, ii) => {
        const v = json.values[idx];
        if (typeof v === 'number' && !isNaN(v)) {
          state[ci][ii].val = Math.max(0, Math.min(100, Math.round(v)));
        }
        idx++;
      }));
      render();
      updateChart();
      statusEl.textContent = '推定しました。内容を確認して調整してください';
    } else {
      statusEl.textContent = 'エラー: ' + (json.error || '推定に失敗しました');
      statusEl.style.color = '#f87171';
    }
  } catch (err) {
    statusEl.textContent = '通信エラー: ' + err;
    statusEl.style.color = '#f87171';
  }
}

function buildStateData() {
  return {
    caseId: document.getElementById('caseId').value,
    grade: document.getElementById('grade').value,
    placement: document.getElementById('placement').value,
    disabilities: Array.from(selectedDisabilities),
    diagnosisText: document.getElementById('diagnosisText').value,
    teacherStyles: Array.from(selectedTeacherStyles),
    teacherNote: document.getElementById('teacherNote').value,
    visitDate: document.getElementById('visitDate').value,
    categories: cats.map((c, ci) => ({
      name: c.name,
      items: c.items.map((it, ii) => ({
        label: it.label,
        value: state[ci][ii].val,
        selectedChips: state[ci][ii].chips.map(x => it.chips[x]),
        note: state[ci][ii].note
      }))
    }))
  };
}

function saveJson() {
  const data = buildStateData();
  const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  const idPart = data.caseId ? data.caseId : 'unnamed';
  a.href = url;
  a.download = `jiritsu_kakuu_${idPart}_${data.visitDate || 'nodate'}.json`;
  a.click();
  document.getElementById('statusMsg').textContent = '保存しました';
  setTimeout(() => { document.getElementById('statusMsg').textContent = ''; }, 2500);
}

let lastGasUrl = DEFAULT_GAS_URL;

async function askAi() {
  const gasUrl = document.getElementById('gasUrl').value.trim();
  lastGasUrl = gasUrl;
  const aiStatus = document.getElementById('aiStatus');
  const aiOutput = document.getElementById('aiOutput');
  if (!gasUrl) {
    aiStatus.textContent = 'GASのURLを入力してください';
    aiStatus.style.color = '#f87171';
    return;
  }
  aiStatus.style.color = '#22d3ee';
  aiStatus.textContent = '相談中です…(数十秒かかることがあります)';
  aiOutput.innerHTML = '';
  try {
    const res = await fetch(gasUrl, { method: 'POST', body: withKey(buildStateData()) });
    const json = await res.json();
    if (json.ok) {
      renderAiOutput(json.summary, json.detail);
      aiStatus.textContent = '完了しました';
    } else {
      aiStatus.textContent = 'エラー: ' + json.error;
      aiStatus.style.color = '#f87171';
    }
  } catch (err) {
    aiStatus.textContent = '通信エラー: ' + err;
    aiStatus.style.color = '#f87171';
  }
}

let lastSummary = '';
let lastDetail = null;

function renderAiOutput(summary, detail) {
  lastSummary = summary;
  lastDetail = detail;
  const aiOutput = document.getElementById('aiOutput');
  aiOutput.innerHTML = '';

  const summaryBox = document.createElement('div');
  summaryBox.style.cssText = 'white-space:pre-wrap; font-size:14px; line-height:1.9; color:#eceef5; margin-top:14px; padding:14px; background:#0e0f14; border:1px solid #7c5cff; border-radius:10px;';
  summaryBox.innerHTML = '<div style="font-size:11px; color:#8b7bff; font-weight:700; margin-bottom:8px;">想定される典型的な支援の要点</div>' + escapeHtml(summary || '');
  aiOutput.appendChild(summaryBox);
  const caution = document.createElement('div');
  caution.style.cssText = 'font-size:11.5px; color:#8b8fa3; margin-top:6px;';
  caution.textContent = '架空の特性から想定した典型例です。AIの出力には誤りが含まれることがあります。';
  aiOutput.appendChild(caution);

  const exportBar = document.createElement('div');
  exportBar.style.cssText = 'display:flex; gap:8px; margin-top:10px;';
  exportBar.innerHTML = `
    <button id="printBtn" style="font-size:11px; padding:6px 14px; border-radius:14px; border:1px solid #333; background:#12141b; color:#a7a9bd; cursor:pointer;">印刷する</button>
    <button id="docBtn" style="font-size:11px; padding:6px 14px; border-radius:14px; border:1px solid #333; background:#12141b; color:#a7a9bd; cursor:pointer;">Googleドキュメントにする</button>
    <span id="docStatus" style="font-size:11px; color:#22d3ee; align-self:center;"></span>
  `;
  aiOutput.appendChild(exportBar);
  document.getElementById('printBtn').onclick = () => window.print();
  document.getElementById('docBtn').onclick = exportToDoc;

  if (!detail) return;

  const toggleBtn = document.createElement('button');
  toggleBtn.className = 'detail-toggle';
  toggleBtn.style.marginTop = '10px';
  toggleBtn.textContent = '詳細版を見る(後日共有用・個別に調整可) ▾';
  const detailBox = document.createElement('div');
  detailBox.style.cssText = 'display:none; margin-top:10px;';
  toggleBtn.onclick = () => {
    const open = detailBox.style.display === 'block';
    detailBox.style.display = open ? 'none' : 'block';
    toggleBtn.textContent = open ? '詳細版を見る(後日共有用・個別に調整可) ▾' : '詳細版を閉じる ▴';
  };
  aiOutput.appendChild(toggleBtn);
  aiOutput.appendChild(detailBox);

  const tendencyBox = document.createElement('div');
  tendencyBox.style.cssText = 'font-size:13px; line-height:1.8; color:#c7c9db; padding:12px 14px; background:#0e0f14; border:1px solid #23242f; border-radius:10px; margin-bottom:12px;';
  tendencyBox.innerHTML = '<div style="font-size:11px; color:#9d9fe8; font-weight:700; margin-bottom:6px;">この特性で典型的に考えられること</div>' + escapeHtml(detail.tendency || '');
  detailBox.appendChild(tendencyBox);

  (detail.goals || []).forEach(goal => renderGoalCard(detailBox, goal));

  if (detail.accommodations) {
    const accBox = document.createElement('div');
    accBox.style.cssText = 'font-size:13px; line-height:1.8; color:#c7c9db; padding:12px 14px; background:#0e0f14; border:1px solid #23242f; border-radius:10px; margin-top:4px;';
    accBox.innerHTML = '<div style="font-size:11px; color:#9d9fe8; font-weight:700; margin-bottom:6px;">合理的配慮・支援機器の候補</div>' + escapeHtml(detail.accommodations);
    detailBox.appendChild(accBox);
  }
}

async function exportToDoc() {
  const docStatus = document.getElementById('docStatus');
  docStatus.textContent = '作成中…';
  try {
    const res = await fetch(lastGasUrl, {
      method: 'POST',
      body: withKey(Object.assign(buildStateData(), {
        action: 'export_doc', summary: lastSummary, detail: lastDetail
      }))
    });
    const json = await res.json();
    if (json.ok) {
      docStatus.innerHTML = `<a href="${json.url}" target="_blank" style="color:#22d3ee;">ドキュメントを開く ↗</a>`;
    } else {
      docStatus.textContent = 'エラー: ' + json.error;
      docStatus.style.color = '#f87171';
    }
  } catch (err) {
    docStatus.textContent = '通信エラー';
    docStatus.style.color = '#f87171';
  }
}

function renderGoalCard(container, goal) {
  const card = document.createElement('div');
  card.style.cssText = 'padding:14px; background:#0e0f14; border:1px solid #23242f; border-radius:10px; margin-bottom:10px;';

  const title = document.createElement('div');
  title.style.cssText = 'font-size:13px; font-weight:700; color:#eceef5; margin-bottom:8px;';
  title.textContent = '目標: ' + goal.title;
  card.appendChild(title);

  const methodsList = document.createElement('ul');
  methodsList.style.cssText = 'margin:0 0 8px; padding-left:18px; font-size:13px; line-height:1.8; color:#c7c9db;';
  card.appendChild(methodsList);

  function addMethodRow(m) {
    const li = document.createElement('li');
    li.style.cssText = 'display:flex; justify-content:space-between; align-items:flex-start; gap:8px;';
    const span = document.createElement('span');
    span.textContent = m;
    const swapBtn = document.createElement('button');
    swapBtn.textContent = '↻別の案に';
    swapBtn.style.cssText = 'font-size:10px; padding:2px 8px; border-radius:10px; border:1px solid #333; background:#12141b; color:#8b7bff; cursor:pointer; white-space:nowrap; flex-shrink:0;';
    swapBtn.onclick = async () => {
      swapBtn.textContent = '…';
      const idx = goal.methods.indexOf(m);
      try {
        const res = await fetch(lastGasUrl, {
          method: 'POST',
          body: withKey(Object.assign(buildStateData(), {
            action: 'more_methods', goalTitle: goal.title,
            existingMethods: goal.methods || [], count: 1
          }))
        });
        const json = await res.json();
        if (json.ok && json.methods && json.methods[0]) {
          const newMethod = json.methods[0];
          if (idx >= 0) goal.methods[idx] = newMethod;
          span.textContent = newMethod;
          m = newMethod;
        }
      } catch (e) { /* noop */ }
      swapBtn.textContent = '↻別の案に';
    };
    li.appendChild(span);
    li.appendChild(swapBtn);
    methodsList.appendChild(li);
  }

  (goal.methods || []).forEach(m => addMethodRow(m));

  if (goal.evaluation) {
    const evalP = document.createElement('div');
    evalP.style.cssText = 'font-size:12px; color:#8b8fa3; margin-bottom:10px;';
    evalP.textContent = '評価の視点: ' + goal.evaluation;
    card.appendChild(evalP);
  }

  const moreBtn = document.createElement('button');
  moreBtn.textContent = '手立てを追加してもらう';
  moreBtn.style.cssText = 'font-size:11px; padding:5px 12px; border-radius:14px; border:1px solid #333; background:#12141b; color:#22d3ee; cursor:pointer; margin-right:6px;';
  moreBtn.onclick = async () => {
    moreBtn.textContent = '提案中…';
    try {
      const res = await fetch(lastGasUrl, {
        method: 'POST',
        body: withKey(Object.assign(buildStateData(), {
          action: 'more_methods', goalTitle: goal.title, existingMethods: goal.methods || [], count: 2
        }))
      });
      const json = await res.json();
      if (json.ok) {
        (json.methods || []).forEach(m => { goal.methods.push(m); addMethodRow(m); });
      }
    } catch (e) { /* noop */ }
    moreBtn.textContent = '手立てを追加してもらう';
  };
  card.appendChild(moreBtn);

  const commentWrap = document.createElement('div');
  commentWrap.style.cssText = 'margin-top:10px;';
  commentWrap.innerHTML = `
    <textarea placeholder="こんな案はどうか、書いて相談できます" style="width:100%; font-size:12px; padding:6px; border:1px solid #2a2c3a; border-radius:6px; background:#14151d; color:#eceef5; resize:vertical; min-height:36px; font-family:inherit;"></textarea>
  `;
  const commentBtn = document.createElement('button');
  commentBtn.textContent = 'この案についてコメントをもらう';
  commentBtn.style.cssText = 'font-size:11px; padding:5px 12px; border-radius:14px; border:1px solid #333; background:#12141b; color:#22d3ee; cursor:pointer; margin-top:6px;';
  const commentOut = document.createElement('div');
  commentOut.style.cssText = 'font-size:12px; color:#c7c9db; line-height:1.7; margin-top:8px; white-space:pre-wrap;';
  commentBtn.onclick = async () => {
    const proposed = commentWrap.querySelector('textarea').value.trim();
    if (!proposed) return;
    commentBtn.textContent = '相談中…';
    try {
      const res = await fetch(lastGasUrl, {
        method: 'POST',
        body: withKey(Object.assign(buildStateData(), {
          action: 'comment_method', goalTitle: goal.title, proposedMethod: proposed
        }))
      });
      const json = await res.json();
      if (json.ok) commentOut.textContent = json.comment;
    } catch (e) { /* noop */ }
    commentBtn.textContent = 'この案についてコメントをもらう';
  };
  commentWrap.appendChild(commentBtn);
  commentWrap.appendChild(commentOut);
  card.appendChild(commentWrap);

  container.appendChild(card);
}

// 架空の児童生徒の例(実在の人物をもとにしていません)
function loadSample() {
  document.getElementById('caseId').value = '架空のAさん';
  document.getElementById('grade').value = '中学部';
  document.getElementById('placement').value = '特別支援学級';
  document.getElementById('diagnosisText').value = '';
  document.getElementById('observationText').value = '移動教室ではいつも一番後ろを歩く。挨拶や受け答えははっきりしていて先生とはよく話すが、友達とは1〜2人としか関わらない。学習には前向きだが、早く終わらせたがる。予定が急に変わると固まってしまうことがある。';
  const pick = (wrapId, set, names) => {
    set.clear();
    document.querySelectorAll('#' + wrapId + ' .chip').forEach(c => {
      const on = names.includes(c.textContent);
      if (on) set.add(c.textContent);
      c.classList.toggle('selected', on);
      c.setAttribute('aria-pressed', String(on));
    });
  };
  pick('disabilityChips', selectedDisabilities, ['自閉スペクトラム症']);
  pick('teacherChips', selectedTeacherStyles, ['視覚的な教材づくり', '観察と記録']);
  const y = new Date(); visitDateInput.value = `${y.getFullYear()}-${String(y.getMonth()+1).padStart(2,'0')}-${String(y.getDate()).padStart(2,'0')}`;
  document.getElementById('estimateStatus').textContent = '架空の例を入れました。「この内容から27項目を推定する」を試せます';
}

function escapeHtml(str) {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
}
</script>
</body>
</html>
