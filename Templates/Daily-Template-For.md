<%*
const input = await tp.system.prompt("날짜 입력 (YYYY-MM-DD)");
const inputDate = moment(input, "YYYY-MM-DD");
if (!inputDate.isValid()) {
  new Notice("올바른 날짜 형식이 아닙니다. (YYYY-MM-DD)");
  return;
}

const fileName = inputDate.format("M월 D일");
await tp.file.rename(fileName);

const today = moment(input);

const allFiles = app.vault.getMarkdownFiles()
  .filter(f => !f.path.startsWith("Templates/"))
  .filter(f => f.basename !== fileName)
  .sort((a, b) => b.stat.mtime - a.stat.mtime);

const files = allFiles.filter(f => {
  const meta = app.metadataCache.getFileCache(f);
  return meta?.frontmatter?.type === "daily-log";
});

let pendingTodos = "";
let debugMsg = "이전 파일 없음";

if (files.length > 0) {
  const prevFile = files[0];
  const prevDate = moment(app.metadataCache.getFileCache(prevFile)?.frontmatter?.date.toString());
  const daysAgo = today.diff(prevDate, "days");
  debugMsg = `${prevFile.basename} (${daysAgo}일 전)`;

  const prevContent = await app.vault.read(prevFile);
  const todos = prevContent
    .split("\n")
    .filter(line => line.match(/^- \[ \]/))
    .join("\n");
  if (todos) pendingTodos = todos;
}
-%>
---
date: <% inputDate.format("YYYY-MM-DD") %>
type: daily-log
---

> [!debug] 이전 로그: <% debugMsg %>

# 📅 <% inputDate.format("M월 D일") %> (<% inputDate.format("dddd") %>)


## ✅ To-Do
> 형식: `- [-] 할 일 내용 [관련링크](URL) #직무태그` / 완료 `[x]` / 포기는 그대로 `[-]`

<% pendingTodos || "- [ ] " %>
