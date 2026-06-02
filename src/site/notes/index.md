---
{"dg-publish":true,"permalink":"/index/","title":"yusquid","dg-note-properties":{"title":"yusquid"}}
---

```dataviewjs
// 【追加】公開サイト用に対策：CSSをここに直接埋め込む
let inlineStyle = `
<style>
/* sizu-cards.css */
.sizu-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr); /* 縦に3列 */
    gap: 24px;
    margin-top: 1rem;
}

/* リンクの下線を消し、カード全体のスタイルを設定 */
a.sizu-card-link {
    text-decoration: none !important;
    display: flex;
    flex-direction: column;
    gap: 12px;
    color: inherit;
    transition: opacity 0.2s ease;
}

a.sizu-card-link:hover {
    opacity: 0.8;
}

/* 外側の横長背景 (#eff7ff) */
.sizu-thumbnail {
    background-color: #eff7ff;
    border-radius: 16px;
    aspect-ratio: 1.5; /* 横長の比率 */
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
    padding: 16px;
    box-sizing: border-box;
}

/* 内側の縦長ドキュメント (#ffffff) */
.sizu-document {
    background-color: #ffffff;
    width: 42%;       /* A4比率に近づける調整 */
    height: 90%;
    border-radius: 4px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.04); /* 微細なリッチな影 */
    padding: 6px 8px;
    overflow: hidden;
    box-sizing: border-box;
}

/* ドキュメント内の最初の数行 (#000000) */
.sizu-preview-text {
    color: #000000;
    font-size: 5px; /* 実際のドキュメント感を出すための極小文字 */
    line-height: 1.6;
    word-wrap: break-word;
    opacity: 0.8;
}

/* 情報エリア（タイトルと日付） */
.sizu-info {
    display: flex;
    flex-direction: column;
    gap: 4px;
    padding: 0 4px;
}

.sizu-title {
    font-size: 1.05em;
    font-weight: 500;
    line-height: 1.4;
    color: var(--text-normal);
    display: -webkit-box;
    -webkit-line-clamp: 2; /* 2行で三点リーダー */
    -webkit-box-orient: vertical;
    overflow: hidden;
}

.sizu-date {
    font-size: 0.85em;
    color: #999999; /* 灰色の小さな日付 */
}

/* スマホ等へのレスポンシブ対応 */
@media (max-width: 768px) {
    .sizu-grid { grid-template-columns: repeat(2, 1fr); }
}
@media (max-width: 480px) {
    .sizu-grid { grid-template-columns: 1fr; }
}
</style>
`;

const targetQuery = '"diary"'; 
const pages = dv.pages(targetQuery).sort(p => p.file.ctime, 'desc').limit(9);

// HTMLの最初にスタイルを結合する
let html = inlineStyle + '<div class="sizu-grid">';

for (let page of pages) {
    let content = await dv.io.load(page.file.path);
    let preview = content.replace(/^---[\s\S]*?---\n/, '').replace(/[#*`_\[\]>]/g, '').trim().substring(0, 100);
    let title = page.file.name;
    let date = page.file.ctime ? page.file.ctime.toFormat('yyyy/M/d') : "";

    html += `
    <a data-href="${page.file.path}" href="${page.file.path}" class="internal-link sizu-card-link">
        <div class="sizu-thumbnail">
            <div class="sizu-document">
                <div class="sizu-preview-text">${preview}</div>
            </div>
        </div>
        <div class="sizu-info">
            <div class="sizu-title">${title}</div>
            <div class="sizu-date">${date}</div>
        </div>
    </a>`;
}
html += '</div>';

dv.container.innerHTML = html;