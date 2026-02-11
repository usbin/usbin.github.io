---
project_name:
update_date:
tags:
depth: update
binary_version:
---


```dataviewjs
  //메타데이터로 리스트업하기
  let pages = dv.pages('"업무/Skywalk/Project/Update/Task"')
	 .where(p => p.update_name == dv.current().file.name); 
  let md = "#### Tasks\n";
  if(pages.length>0){
	pages.forEach(p=>{
		md += `- ${p.file.link}${p.status=="Completed"?"✅":"❗"}\n`;
	});
  }
  else{
	md+= "- empty";
  }
  dv.paragraph(md);
  
```