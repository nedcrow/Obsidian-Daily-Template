<%*
await tp.file.rename(tp.date.now("M월 D일"));

const currentFileName = tp.date.now("M월 D일");
const today = moment(tp.date.now("YYYY-MM-DD"));

const allFiles = app.vault.getMarkdownFiles()
  .filter(f => !f.path.startsWith("Templates/"))
  .filter(f => f.basename !== currentFileName)
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
date: <% tp.date.now("YYYY-MM-DD") %>
type: daily-log
---

> [!debug] 이전 로그: <% debugMsg %>

# 📅 <% tp.date.now("M월 D일") %> (<% tp.date.now("dddd") %>)


## ✅ To-Do
> 형식: `- [-] 할 일 내용 [관련링크](URL) #직무태그` / 완료 `[x]` / 포기는 그대로 `[-]`

<% pendingTodos || "- [ ] " %>
