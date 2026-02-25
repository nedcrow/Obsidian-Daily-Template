---
state:
  showOnlyLogged: false
---
# 🌱 2026년 잔디

```dataviewjs
const page = dv.current();
const file = app.workspace.getActiveFile(); // ✅ 추가

const folderName = page.file.folder.split("/").pop();
const year = parseInt(folderName.match(/\d+/)[0]);
const today = moment().startOf("day");

// ✅ namespace 기준으로 읽기
let showOnlyLogged = page.state?.showOnlyLogged ?? false;

// 토글 버튼
const toggleBtn = this.container.createEl("button", {
  text: `📋 기록된 날만 보기: ${showOnlyLogged ? "ON" : "OFF"}`,
  attr: { 
    style: "margin-bottom:12px; padding:4px 12px; cursor:pointer; border-radius:6px; border:1px solid #888; background:#333; color:#fff;" 
  }
});

// 초기 색상 반영
toggleBtn.style.background = showOnlyLogged ? "#4caf50" : "#333";

toggleBtn.addEventListener("click", async () => {
  showOnlyLogged = !showOnlyLogged;

  toggleBtn.setText(`📋 기록된 날만 보기: ${showOnlyLogged ? "ON" : "OFF"}`);
  toggleBtn.style.background = showOnlyLogged ? "#4caf50" : "#333";

  // ✅ frontmatter 저장
  await app.fileManager.processFrontMatter(file, fm => {  
    if (!fm.state) fm.state = {};  
    fm.state.showOnlyLogged = showOnlyLogged;  
  });

  renderGrass();
});

// 파일 읽기
const files = dv.pages()
  .where(p => p.type === "daily-log" && p.date && moment(p.date.toString()).year() === year)
  .array();

const completedDates = new Set();
const loggedDates = new Set();

for (const p of files) {
  const f = app.vault.getAbstractFileByPath(p.file.path);
  if (!f) continue;

  const content = await app.vault.read(f);
  const dateStr = moment(p.date.toString()).format("YYYY-MM-DD");
  loggedDates.add(dateStr);

  const hasIncomplete = content.split("\n")
    .filter(line => !line.trimStart().startsWith(">"))
    .some(line => line.match(/^- \[ \]/));

  if (!hasIncomplete) completedDates.add(dateStr);
}

// 렌더링
const grassContainer = this.container.createEl("div", {
  attr: { style: "display:flex; flex-wrap:wrap; gap:24px;" }
});
const weekdays = ["일", "월", "화", "수", "목", "금", "토"];

function renderGrass() {
  grassContainer.empty();

  for (let month = 1; month <= 12; month++) {
    const monthMoment = moment(`${year}-${String(month).padStart(2, "0")}-01`);
    if (monthMoment.isAfter(today, "month")) continue;

    const monthEl = grassContainer.createEl("div", { attr: { style: "margin-bottom:16px;" } });
    monthEl.createEl("h3", { text: `${month}월`, attr: { style: "margin-bottom:6px;" } });

    const grid = monthEl.createEl("div", {
      attr: { style: "display:grid; grid-template-columns: repeat(7, 24px); gap:4px;" }
    });

    if (!showOnlyLogged) {
      weekdays.forEach((d, i) => {
        grid.createEl("div", {
          text: d,
          attr: {
            style: `width:24px; height:24px; text-align:center; line-height:24px; font-size:11px;
              color:${i === 0 ? "#e57373" : i === 6 ? "#64b5f6" : "#aaa"};`
          }
        });
      });

      const firstDay = monthMoment.day();
      for (let i = 0; i < firstDay; i++) {
        grid.createEl("div", { attr: { style: "width:24px; height:24px;" } });
      }
    }

    const daysInMonth = monthMoment.daysInMonth();
    for (let day = 1; day <= daysInMonth; day++) {
      const dateMoment = moment(`${year}-${String(month).padStart(2, "0")}-${String(day).padStart(2, "0")}`);
      if (dateMoment.isAfter(today)) break;

      const dateStr = dateMoment.format("YYYY-MM-DD");
      if (showOnlyLogged && !loggedDates.has(dateStr)) continue;

      const isCompleted = completedDates.has(dateStr);
      const isLogged = loggedDates.has(dateStr);
      const isToday = dateMoment.isSame(today, "day");
      const dayOfWeek = dateMoment.day();

      const cell = grid.createEl("div", {
        attr: {
          style: `width:24px; height:24px; border-radius:4px; text-align:center; line-height:24px; font-size:11px; cursor:pointer;
            background:${isCompleted ? "#4caf50" : isLogged ? "#888" : "#444"};
            border:${isToday ? "2px solid #fff" : "none"};
            color:${isLogged ? "#fff" : dayOfWeek === 0 ? "#e57373" : dayOfWeek === 6 ? "#64b5f6" : "#666"};`
        },
        title: dateStr
      });
      cell.setText(String(day));
    }
  }
}

renderGrass();
```
