---
{"dg-publish":true,"permalink":"/diary-index/","title":"yusquid","dg-note-properties":{"title":"yusquid"}}
---


## 戦闘記録

```dataviewjs
const pages = dv.pages('"yusquid"')
  .where(p => p.date)
  .sort(p => p.date, 'desc');

let html = '<div class="battle-cards">';

for (const page of pages) {
  html += `
    <a class="battle-card" href="${page.file.path.replace('.md','')}">
      <h3>${page.file.name}</h3>
      <p>${page.date}</p>
    </a>
  `;
}

html += '</div>';

dv.paragraph(html);
```