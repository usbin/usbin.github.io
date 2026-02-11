```dataviewjs

// 탐색할 최상위 폴더 경로
let rootFolder = "업무/Skywalk/보관함/걸글로브 서버";
let allPages = dv.pages(`"${rootFolder}"`);

// 현재 폴더에 속한 페이지 가져오기
function pagesInFolder(folderPath) {
    return allPages.where(p => p.file.folder === folderPath);
}

// 현재 폴더의 직속 하위 폴더
function subFolders(folderPath) {
    let subs = new Set();
    allPages.forEach(p => {
        if (p.file.folder.startsWith(folderPath + "/")) {
            let relative = p.file.folder.slice(folderPath.length + 1);
            let first = relative.split("/")[0];
            if (first) subs.add(first);
        }
    });
    return Array.from(subs).sort();
}

// 재귀적으로 Markdown 리스트 생성
function renderTree(folderPath, level=0) {
    let md = "";
    let prefix = "  ".repeat(level); // 들여쓰기

    // 파일
    pagesInFolder(folderPath).forEach(p => {
        md += `${prefix}- [[${p.file.name}]]\n`;
    });

    // 하위 폴더
    subFolders(folderPath).forEach(sub => {
        md += `${prefix}- ${sub}\n`;
        md += renderTree(folderPath + "/" + sub, level + 1);
    });

    return md;
}

// 렌더링
dv.paragraph(renderTree(rootFolder));
```