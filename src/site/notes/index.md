---
{"dg-publish":true,"permalink":"/index/","title":"yusquid","dg-note-properties":{"title":"yusquid"}}
---

```dataviewjs

{

    // ----------------------------------------------------

    // 1. CSSの統合（改行によるレイアウト崩れ対策を強化）

    // ----------------------------------------------------

    let combinedStyle = `

    <style>

    /* Heatmap Calendar用 */

    .heatmap-calendar-graph rect[data-level="0"],

    .heatmap-calendar-graph rect[fill="transparent"],

    .heatmap-calendar-graph .color-empty {

        fill: #ecf1f4 !important;

    }

  

    /* Sizu風カード用 */

    .sizu-grid {

        display: grid !important;

        grid-template-columns: repeat(3, 1fr) !important;

        gap: 24px !important;

        margin-bottom: 3rem; /* カレンダーとの余白 */

        width: 100%;

    }

    a.sizu-card-link {

        text-decoration: none !important;

        display: flex !important;

        flex-direction: column !important;

        gap: 12px !important;

        color: inherit !important;

        transition: opacity 0.2s ease;

    }

    a.sizu-card-link:hover {

        opacity: 0.8;

    }

    .sizu-thumbnail {

        background-color: #eff7ff;

        border-radius: 16px;

        aspect-ratio: 1.5;

        display: flex;

        align-items: center;

        justify-content: center;

        position: relative;

        padding: 16px;

        box-sizing: border-box;

    }

    .sizu-document {

        background-color: #ffffff;

        width: 42%;

        height: 90%;

        border-radius: 4px;

        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.04);

        padding: 6px 8px;

        overflow: hidden;

        box-sizing: border-box;

    }

    .sizu-preview-text {

        color: #000000;

        font-size: 5px;

        line-height: 1.6;

        word-wrap: break-word;

        opacity: 0.8;

    }

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

        -webkit-line-clamp: 2;

        -webkit-box-orient: vertical;

        overflow: hidden;

    }

    .sizu-date {

        font-size: 0.85em;

        color: #999999;

    }

    @media (max-width: 768px) {

        .sizu-grid { grid-template-columns: repeat(2, 1fr) !important; }

    }

    @media (max-width: 480px) {

        .sizu-grid { grid-template-columns: 1fr !important; }

    }

    /* カレンダーの間隔用 */

    .calendar-wrapper {

        margin-bottom: 2rem;

    }

    </style>

    `;

  

    // コンテナを一旦クリアしてスタイルを適用

    dv.container.innerHTML = combinedStyle;

  

    // ----------------------------------------------------

    // 2. Sizu風カードグリッド の描画エリア（一番上）

    // ----------------------------------------------------

    const cardsContainer = dv.el("div", "");

    const targetQuery = '""'; 

    const latestPages = dv.pages(targetQuery).sort(p => p.file.ctime, 'desc').limit(9);

  

    let html = '<div class="sizu-grid">';

  

    for (let page of latestPages) {

        let content = await dv.io.load(page.file.path);

        let preview = content.replace(/^---[\s\S]*?---\n/, '').replace(/[#*`_\[\]>]/g, '').trim().substring(0, 100);

        let title = page.file.name;

        let date = page.file.ctime ? page.file.ctime.toFormat('yyyy/M/d') : "";

  

        // ★重要★: Obsidianの誤変換を防ぐため、HTMLタグ内の改行を削除して1行にしています

        html += `<a data-href="${page.file.path}" href="${page.file.path}" class="internal-link sizu-card-link"><div class="sizu-thumbnail"><div class="sizu-document"><div class="sizu-preview-text">${preview}</div></div></div><div class="sizu-info"><div class="sizu-title">${title}</div><div class="sizu-date">${date}</div></div></a>`;

    }

    html += '</div>';

    // 生成したカードHTMLを流し込む

    cardsContainer.innerHTML = html;

  

  

    // ----------------------------------------------------

    // 3. Heatmap Calendar の描画エリア（カードの下）

    // ----------------------------------------------------

    const folderName = '"diary"';

    const diaryPages = dv.pages(folderName).where(p => p.date);

  

    // カレンダーを生成する共通関数

    function drawCalendar(targetYear) {

        // カレンダーごとに専用のdiv枠を作る

        const yearContainer = document.createElement("div");

        yearContainer.addClass("calendar-wrapper");

        dv.container.appendChild(yearContainer);

  

        const dateCounts = {};

        for (let page of diaryPages) {

            let d = page.date;

            let parsedDate = typeof d === 'string' ? dv.date(d) : d;

            if (parsedDate && parsedDate.year === targetYear) {

                let dateStr = parsedDate.toFormat("yyyy-MM-dd");

                dateCounts[dateStr] = (dateCounts[dateStr] || 0) + 1;

            }

        }

  

        const entries = [];

        for (let [dateStr, count] of Object.entries(dateCounts)) {

            entries.push({

                date: dateStr,

                intensity: count >= 2 ? 2 : 1

            });

        }

  

        const calendarData = {

            year: targetYear,

            colors: {

                theme: ["#6ae583", "#2fbd4b"]

            },

            entries: entries,

            showCurrentDayBorder: false

        };

  

        // カレンダーの描画

        renderHeatmapCalendar(yearContainer, calendarData);

    }

  

    // 上から順に表示したい年を指定する（2026年 → 2025年の順）

    drawCalendar(2026);

    drawCalendar(2025);

}