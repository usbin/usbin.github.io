---
active: true
depth: project
---

```dataviewjs
   function getUpdateDatePrintFormat(update){
   	return update ? update.update_date? update.update_date.toFormat("yyyy-MM-dd") : "미정" : "미정";
   }
   let now = dv.luxon.DateTime.now();
     //메타데이터로 리스트업하기
     let pages = dv.pages('"업무/Skywalk/Project/Update"')
   	 .where(p => p.project_name == dv.current().file.name && p.depth == "update")
   	 .sort(p=>p.update_date, 'desc'); 
     let md = "#### Updates\n";
     if(pages.length>0){
   	pages.forEach(p=>{
   		md += `- ${p.file.link} (${getUpdateDatePrintFormat(p)}) ${p.update_date< now ?"✅":"❗"}\n`;
   	});
     }
     else{
   	md+= "- empty";
     }
     dv.paragraph(md);
     
   ```