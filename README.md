[index.html](https://github.com/user-attachments/files/28942562/index.html)
<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>🏗️ 工程管理系統 Pro</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.2/babel.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    html, body, #root { height: 100%; }
    body { font-family: system-ui, -apple-system, "Segoe UI", "PingFang TC", "Microsoft JhengHei", sans-serif; background: #f8fafc; overflow: hidden; font-weight: 500; }
    @media (max-width: 767px) {
      html, body, #root { height: auto; overflow: visible; }
      body { overflow-x: hidden; overflow-y: auto; }
    }
    ::-webkit-scrollbar { width: 5px; height: 5px; }
    ::-webkit-scrollbar-track { background: #f1f5f9; }
    ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 3px; }
    input[type="date"]::-webkit-calendar-picker-indicator { cursor: pointer; opacity: 0.45; }
    button:active { transform: scale(.97); }
    .t-row:hover { background: #f0f7ff !important; }
  </style>
</head>
<body>
<div id="root"></div>
<script type="text/babel">
const { useState, useMemo, useEffect } = React;
const DAY_PX=22, LABEL_W=148, BP_H=28;
const COLORS=["#2563eb","#0d9488","#b45309","#7c3aed","#0e7490","#be185d","#dc2626","#4d7c0f"];
const WEEKDAY="日一二三四五六";
const S_STYLE={
  "待執行":{bg:"#f1f5f9",color:"#64748b",border:"#cbd5e1"},
  "執行中":{bg:"#eff6ff",color:"#2563eb",border:"#93c5fd"},
  "已完成":{bg:"#f0fdf4",color:"#16a34a",border:"#86efac"},
};
const TW_HOLIDAYS=new Map([
  ["2025-01-01","元旦"],["2025-01-27","農曆除夕"],["2025-01-28","春節"],["2025-01-29","春節"],
  ["2025-01-30","春節"],["2025-01-31","春節"],["2025-02-28","和平紀念日"],
  ["2025-04-04","児童節"],["2025-04-05","清明節"],["2025-05-01","勞動節"],
  ["2025-05-31","端午節"],["2025-10-06","中秋節"],["2025-10-10","國慶日"],
  ["2026-01-01","元旦"],["2026-02-16","農曆除夕"],["2026-02-17","春節"],["2026-02-18","春節"],
  ["2026-02-19","春節"],["2026-02-20","春節"],["2026-02-28","和平紀念日"],
  ["2026-04-04","児童節"],["2026-04-05","清明節"],["2026-05-01","勞動節"],
  ["2026-06-19","端午節"],["2026-09-25","中秋節"],["2026-10-10","國慶日"],
]);
const dateKey=d=>`${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,"0")}-${String(d.getDate()).padStart(2,"0")}`;
const isHoliday=d=>TW_HOLIDAYS.has(dateKey(d));
const holidayName=d=>TW_HOLIDAYS.get(dateKey(d))??"";
const parseDate=s=>new Date(s+"T00:00:00");
const diffDays=(a,b)=>Math.round((b-a)/86400000);
const daysInMonth=(y,m)=>new Date(y,m+1,0).getDate();
const parseN=s=>{const n=String(s||"").replace(/[^0-9]/g,"");return n?Number(n):0;};
const fmtN=n=>Number(n).toLocaleString("zh-TW");
const calcStatus=t=>{
  if(t.done)return"已完成";
  const today=new Date();today.setHours(0,0,0,0);
  return today<parseDate(t.startDate)?"待執行":"執行中";
};
const calcWorkDays=(s0,e0)=>{
  const s=parseDate(s0),e=parseDate(e0);let c=0,cur=new Date(s);
  while(cur<=e){const d=cur.getDay();if(d!==0&&d!==6&&!isHoliday(cur))c++;cur.setDate(cur.getDate()+1);}
  return c;
};
const uid=()=>Math.random().toString(36).slice(2,9);
const mkDetail=()=>({estimatedCost:"",actualCost:"",vendor:"",content:"",tracking:"",payment:"",paymentImages:[]});
const mkSteps=()=>({
  stamp:{sent:false,confirmed:false},
  contactDeposit:{invoiceReceived:false,confirmedDate:"",remittanceDate:"",amount:"",percent:""},
  contactFinal:{invoiceReceived:false,confirmedDate:"",remittanceDate:""},
  contract:{signed:false,internalSeal:false,vendorSeal:false,archived:false},
  plan:{item1:"",item2:"",item3:""},
});
const today0=()=>new Date().toISOString().slice(0,10);
const after30=()=>new Date(Date.now()+30*86400000).toISOString().slice(0,10);
const mkTask=(name="新工程",s=today0(),e=after30())=>({
  id:uid(),name,startDate:s,endDate:e,done:false,taskBudget:"",detail:mkDetail(),steps:mkSteps()
});
const mkBigProject=(name="新大專案")=>({
  id:uid(),name,totalBudget:"",collapsed:false,tasks:[mkTask("工程 1")]
});
const useWindowWidth=()=>{
  const[w,setW]=useState(window.innerWidth);
  useEffect(()=>{const h=()=>setW(window.innerWidth);window.addEventListener("resize",h);return()=>window.removeEventListener("resize",h);},[]);
  return w;
};
function ProjectDetailForm({t,setDetailField,color}){
  const det=t.detail||{};
  const addImages=files=>Array.from(files).forEach(file=>{
    const r=new FileReader();
    r.onload=ev=>setDetailField(t.id,"paymentImages",[...(t.detail?.paymentImages||[]),{name:file.name,data:ev.target.result}]);
    r.readAsDataURL(file);
  });
  const removeImg=i=>setDetailField(t.id,"paymentImages",(t.detail?.paymentImages||[]).filter((_,j)=>j!==i));
  const fieldDefs=[
    {key:"actualCost",label:"實際費用（含稅，元）",type:"numfmt",icon:"💳"},
    {key:"vendor",label:"配合廠商",type:"text",icon:"🏢"},
    {key:"content",label:"工程內容",type:"ta",icon:"📝"},
    {key:"tracking",label:"待追蹤事項",type:"ta",icon:"🔍"},
    {key:"payment",label:"匯款資訊",type:"pay",icon:"🏦"},
  ];
  const inp={border:"1px solid #e2e8f0",borderRadius:4,padding:"6px 10px",fontSize:12,outline:"none",fontFamily:"inherit",width:"100%"};
  const fo=(e,c)=>{e.target.style.borderColor=c;e.target.style.boxShadow=`0 0 0 2px ${c}22`;};
  const bl=e=>{e.target.style.borderColor="#e2e8f0";e.target.style.boxShadow="none";};
  return(
    <div>
      <div style={{fontSize:12,fontWeight:700,color:"#475569",marginBottom:12,paddingBottom:6,borderBottom:"2px solid "+color}}>🗂️ 專案詳情</div>
      {fieldDefs.map(({key,label,type,icon})=>(
        <div key={key} style={{marginBottom:12}}>
          <div style={{fontSize:11,fontWeight:700,color:"#64748b",marginBottom:4}}>{icon} {label}</div>
          {type==="ta"&&<textarea rows={3} value={det[key]??""} onClick={e=>e.stopPropagation()}
            onChange={e=>{e.stopPropagation();setDetailField(t.id,key,e.target.value);}}
            style={{...inp,resize:"vertical",minHeight:60}} onFocus={e=>fo(e,color)} onBlur={bl}/>}
          {type==="numfmt"&&<input type="text" inputMode="numeric" value={det[key]??""} onClick={e=>e.stopPropagation()}
            onChange={e=>{e.stopPropagation();const n=String(e.target.value).replace(/[^0-9]/g,"");setDetailField(t.id,key,n?Number(n).toLocaleString("zh-TW"):"");}}
            style={{...inp,textAlign:"right"}} onFocus={e=>fo(e,color)} onBlur={bl} placeholder="0"/>}
          {type==="text"&&<input type="text" value={det[key]??""} onClick={e=>e.stopPropagation()}
            onChange={e=>{e.stopPropagation();setDetailField(t.id,key,e.target.value);}}
            style={inp} onFocus={e=>fo(e,color)} onBlur={bl}/>}
          {type==="pay"&&<div>
            <textarea rows={2} value={det[key]??""} placeholder="備註匯款帳號、日期等…" onClick={e=>e.stopPropagation()}
              onChange={e=>{e.stopPropagation();setDetailField(t.id,key,e.target.value);}}
              style={{...inp,resize:"vertical",minHeight:50,marginBottom:8}} onFocus={e=>fo(e,color)} onBlur={bl}/>
            <label style={{display:"inline-flex",alignItems:"center",gap:5,cursor:"pointer",background:color+"15",border:`1px dashed ${color}55`,borderRadius:4,padding:"5px 12px",fontSize:11,color,fontWeight:700}} onClick={e=>e.stopPropagation()}>
              📎 上傳憑證照片
              <input type="file" accept="image/*" multiple style={{display:"none"}} onChange={e=>{e.stopPropagation();addImages(e.target.files);e.target.value="";}}/>
            </label>
            {(det.paymentImages||[]).map((img,i)=>(
              <div key={i} style={{display:"flex",alignItems:"center",gap:8,marginTop:7,padding:"6px 8px",background:"#f8fafc",borderRadius:4,border:"1px solid #e2e8f0"}}>
                <img src={img.data} style={{width:52,height:42,objectFit:"cover",borderRadius:3,flexShrink:0,cursor:"pointer"}} onClick={e=>{e.stopPropagation();window.open(img.data,"_blank");}} title="點擊放大"/>
                <span style={{flex:1,fontSize:11,color:"#475569",overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>{img.name}</span>
                <a href={img.data} download={img.name} title="下載" style={{fontSize:17,textDecoration:"none",lineHeight:1,flexShrink:0}} onClick={e=>e.stopPropagation()}>⬇️</a>
                <button onClick={e=>{e.stopPropagation();removeImg(i);}} style={{background:"none",border:"none",cursor:"pointer",color:"#fca5a5",fontSize:16,padding:0,lineHeight:1,flexShrink:0}}
                  onMouseEnter={e=>e.currentTarget.style.color="#ef4444"} onMouseLeave={e=>e.currentTarget.style.color="#fca5a5"}>✕</button>
              </div>
            ))}
          </div>}
        </div>
      ))}
    </div>
  );
}
function DetailPanel({t,color,setStepField}){
  const grp={marginBottom:10};
  const grpLabel={fontSize:11,fontWeight:700,color:"#334155",marginBottom:5,display:"flex",alignItems:"center",gap:4};
  const subLabel={fontSize:10,fontWeight:700,color:"#64748b",marginBottom:4,paddingLeft:5,borderLeft:"2px solid #cbd5e1"};
  const Chk=({label,path,val})=>(
    <label style={{display:"flex",alignItems:"center",gap:4,cursor:"pointer",fontSize:11,color:"#334155"}} onClick={e=>e.stopPropagation()}>
      <input type="checkbox" checked={!!val} onChange={()=>setStepField(t.id,path,!val)} style={{accentColor:color,cursor:"pointer",width:12,height:12}}/>
      <span>{label}</span>
    </label>
  );
  const DateRow=({label,path,val})=>(
    <div style={{display:"flex",alignItems:"center",gap:5,marginBottom:3,paddingLeft:2}}>
      <span style={{fontSize:10,color:"#94a3b8",width:50,flexShrink:0}}>{label}</span>
      <input type="date" value={val} onChange={e=>setStepField(t.id,path,e.target.value)} onClick={e=>e.stopPropagation()}
        style={{border:`1px solid ${val?color:"#cbd5e1"}`,borderRadius:3,padding:"2px 4px",fontSize:10,outline:"none",color:val?color:"#94a3b8",background:val?color+"10":"white",cursor:"pointer",fontFamily:"inherit"}}/>
      {val&&<span style={{fontSize:10,color}}>✓</span>}
    </div>
  );
  const NumItem=({n,path,val})=>(
    <div style={{display:"flex",alignItems:"center",gap:5,marginBottom:3}} onClick={e=>e.stopPropagation()}>
      <span style={{fontSize:11,color:"#94a3b8",flexShrink:0}}>{n}.</span>
      <input type="text" placeholder="填寫內容" value={val||""} onChange={e=>setStepField(t.id,path,e.target.value)}
        style={{border:"1px solid #cbd5e1",borderRadius:3,padding:"3px 6px",fontSize:10,outline:"none",fontFamily:"inherit",width:"100%"}}
        onFocus={e=>e.target.style.borderColor=color} onBlur={e=>e.target.style.borderColor="#cbd5e1"}/>
    </div>
  );
  const DepositRow=()=>{
    const evalExpr=str=>{const s=String(str).slice(1).replace(/[^0-9+\-*/().]/g,"");try{const r=Function('"use strict";return('+s+")")();return isFinite(r)?Math.round(r):null;}catch{return null;}};
    const extractBase=str=>{const m=String(str).slice(1).match(/^(\d+)/);return m?Number(m[1]):null;};
    const actNum=()=>{const n=String(t.detail?.actualCost||"").replace(/[^0-9]/g,"");return n?Number(n):null;};
    const onAmtChange=e=>{
      e.stopPropagation();const v=e.target.value;
      if(v.startsWith("=")){setStepField(t.id,"contactDeposit.amount",v);return;}
      const n=v.replace(/[^0-9]/g,"");
      setStepField(t.id,"contactDeposit.amount",n?fmtN(Number(n)):"");
      if(!n){setStepField(t.id,"contactDeposit.percent","");return;}
      const act=actNum();if(act)setStepField(t.id,"contactDeposit.percent",((Number(n)/act)*100).toFixed(1));
    };
    const onAmtBlur=()=>{
      const v=t.steps.contactDeposit.amount||"";if(!v.startsWith("="))return;
      const r=evalExpr(v);if(r===null)return;
      const base=extractBase(v)||actNum();
      setStepField(t.id,"contactDeposit.amount",fmtN(r));
      if(base)setStepField(t.id,"contactDeposit.percent",((r/base)*100).toFixed(1));
    };
    const onPctChange=e=>{
      e.stopPropagation();const p=e.target.value.replace(/[^0-9.]/g,"");
      setStepField(t.id,"contactDeposit.percent",p);
      const act=actNum();if(p&&act)setStepField(t.id,"contactDeposit.amount",fmtN(Math.round(act*Number(p)/100)));
    };
    const iS={border:"1px solid #cbd5e1",borderRadius:3,padding:"3px 6px",fontSize:10,outline:"none",textAlign:"right",fontFamily:"inherit"};
    return(
      <div style={{display:"flex",gap:6,marginBottom:6,alignItems:"center"}}>
        <div style={{display:"flex",alignItems:"center",gap:3,flex:1}}>
          <input type="text" placeholder="金額 或 =運算式" value={t.steps.contactDeposit.amount??""} onClick={e=>e.stopPropagation()}
            onChange={onAmtChange} onBlur={onAmtBlur}
            onKeyDown={e=>{if(e.key==="Enter"){e.stopPropagation();onAmtBlur();e.target.blur();}}}
            style={{...iS,width:"100%",color:(t.steps.contactDeposit.amount||"").startsWith("=")?"\#7c3aed":"inherit"}}
            onFocus={e=>e.target.style.borderColor=color}/>
          <span style={{fontSize:10,color:"#64748b",flexShrink:0,whiteSpace:"nowrap"}}>元（含稅）</span>
        </div>
        <div style={{display:"flex",alignItems:"center",gap:3,width:72}}>
          <input type="text" inputMode="numeric" placeholder="%" value={t.steps.contactDeposit.percent??""} onClick={e=>e.stopPropagation()}
            onChange={onPctChange} style={{...iS,width:"100%"}}
            onFocus={e=>e.target.style.borderColor=color} onBlur={e=>e.target.style.borderColor="#cbd5e1"}/>
          <span style={{fontSize:10,color:"#64748b",flexShrink:0}}>%</span>
        </div>
      </div>
    );
  };
  return(
    <>
      <div style={grp}>
        <div style={grpLabel}>📄 簽呈</div>
        <div style={{paddingLeft:6,display:"flex",gap:14}}>
          <Chk label="已送出" path="stamp.sent" val={t.steps.stamp.sent}/>
          <Chk label="已查照" path="stamp.confirmed" val={t.steps.stamp.confirmed}/>
        </div>
      </div>
      <div style={grp}>
        <div style={grpLabel}>💰 聯絡單</div>
        <div style={{paddingLeft:6,marginBottom:6}}>
          <div style={subLabel}>（訂金）</div>
          <div style={{paddingLeft:6}}>
            <DepositRow/>
            <div style={{marginBottom:4}}><Chk label="已收到發票" path="contactDeposit.invoiceReceived" val={t.steps.contactDeposit.invoiceReceived}/></div>
            <DateRow label="查照完成" path="contactDeposit.confirmedDate" val={t.steps.contactDeposit.confirmedDate}/>
            <DateRow label="匯款" path="contactDeposit.remittanceDate" val={t.steps.contactDeposit.remittanceDate}/>
          </div>
        </div>
        <div style={{paddingLeft:6}}>
          <div style={subLabel}>（尾款）</div>
          <div style={{paddingLeft:6}}>
            <div style={{marginBottom:4}}><Chk label="已收到發票" path="contactFinal.invoiceReceived" val={t.steps.contactFinal.invoiceReceived}/></div>
            <DateRow label="查照完成" path="contactFinal.confirmedDate" val={t.steps.contactFinal.confirmedDate}/>
            <DateRow label="匯款" path="contactFinal.remittanceDate" val={t.steps.contactFinal.remittanceDate}/>
          </div>
        </div>
      </div>
      <div style={grp}>
        <div style={grpLabel}>📋 合約</div>
        <div style={{paddingLeft:6,display:"flex",flexDirection:"column",gap:5}}>
          <Chk label="已簽署" path="contract.signed" val={t.steps.contract.signed}/>
          <Chk label="內部用印" path="contract.internalSeal" val={t.steps.contract.internalSeal}/>
          <Chk label="廠商用印" path="contract.vendorSeal" val={t.steps.contract.vendorSeal}/>
          <Chk label="已歸檔" path="contract.archived" val={t.steps.contract.archived}/>
        </div>
      </div>
      <div style={grp}>
        <div style={grpLabel}>📐 其他</div>
        <div style={{paddingLeft:6,display:"flex",flexDirection:"column",gap:5}}>
          <NumItem n={1} path="plan.item1" val={t.steps.plan.item1}/>
          <NumItem n={2} path="plan.item2" val={t.steps.plan.item2}/>
          <NumItem n={3} path="plan.item3" val={t.steps.plan.item3}/>
        </div>
      </div>
    </>
  );
}
function DrawerPanel({taskInfo,drawerTab,setDrawerTab,setDrawerTaskId,setDetailField,setStepField}){
  const{t,color}=taskInfo;
  return(
    <div style={{display:"flex",flexDirection:"column",height:"100%"}}>
      <div style={{background:color,color:"white",padding:"10px 14px",display:"flex",alignItems:"center",justifyContent:"space-between",flexShrink:0}}>
        <span style={{fontWeight:700,fontSize:13,overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap",maxWidth:"calc(100% - 36px)"}}>📋 {t.name}</span>
        <button onClick={()=>setDrawerTaskId(null)} style={{background:"none",border:"none",color:"white",fontSize:19,cursor:"pointer",lineHeight:1,padding:0}}>×</button>
      </div>
      <div style={{display:"flex",borderBottom:"1px solid #e2e8f0",background:"#f8fafc",flexShrink:0}}>
        {[["detail","🗂️ 專案詳情"],["checklist","✅ 確認表"]].map(([tab,label])=>(
          <button key={tab} onClick={e=>{e.stopPropagation();setDrawerTab(tab);}}
            style={{flex:1,padding:"8px 0",fontSize:12,fontWeight:700,cursor:"pointer",border:"none",
              borderBottom:drawerTab===tab?`2px solid ${color}`:"2px solid transparent",
              background:"transparent",color:drawerTab===tab?color:"#64748b"}}>
            {label}
          </button>
        ))}
      </div>
      <div style={{flex:1,overflowY:"auto",padding:14}}>
        {drawerTab==="detail"
          ?<ProjectDetailForm t={t} setDetailField={setDetailField} color={color}/>
          :<div>
            <div style={{fontSize:10,fontWeight:700,color,background:color+"18",display:"inline-block",padding:"1px 7px",borderRadius:10,marginBottom:12}}>✅ 確認表</div>
            {DetailPanel({t,color,setStepField})}
          </div>
        }
      </div>
    </div>
  );
}
function App(){
  const STORAGE_KEY="engMgmtProData";
  const defaultProjects=()=>[
    {id:uid(),name:"台北信義案",totalBudget:"",collapsed:false,tasks:[
      mkTask("水電配管","2026-06-01","2026-07-15"),
      mkTask("泥作工程","2026-07-01","2026-08-31"),
      mkTask("油漆收尾","2026-08-15","2026-09-30"),
    ]},
    {id:uid(),name:"新竹竹北案",totalBudget:"",collapsed:false,tasks:[
      mkTask("設計規劃","2026-06-10","2026-07-31"),
      mkTask("結構工程","2026-07-15","2026-09-30"),
    ]},
  ];
  const[bigProjects,setBigProjects]=useState(()=>{
    try{
      const saved=localStorage.getItem(STORAGE_KEY);
      if(saved){const parsed=JSON.parse(saved);if(Array.isArray(parsed)&&parsed.length)return parsed;}
    }catch(e){}
    return defaultProjects();
  });
  useEffect(()=>{
    try{localStorage.setItem(STORAGE_KEY,JSON.stringify(bigProjects));}catch(e){}
  },[bigProjects]);
  // ===== 雲端同步（Google 試算表 + Apps Script）=====
  const CLOUD_KEY="engMgmtCloudUrl";
  const[cloudUrl,setCloudUrl]=useState(()=>{try{return localStorage.getItem(CLOUD_KEY)||"";}catch(e){return"";}});
  const[showCloudSettings,setShowCloudSettings]=useState(false);
  const[cloudUrlInput,setCloudUrlInput]=useState(cloudUrl);
  const[syncStatus,setSyncStatus]=useState("idle"); // idle | loading | syncing | synced | error
  const skipNextSave=React.useRef(true);
  const lastJsonRef=React.useRef("");
  const saveCloudUrl=()=>{
    const u=cloudUrlInput.trim();
    try{
      if(u)localStorage.setItem(CLOUD_KEY,u);else localStorage.removeItem(CLOUD_KEY);
    }catch(e){}
    skipNextSave.current=true;
    setCloudUrl(u);
    setShowCloudSettings(false);
  };
  // 開啟頁面 / 設定網址後，從雲端讀取資料
  useEffect(()=>{
    if(!cloudUrl)return;
    setSyncStatus("loading");
    fetch(cloudUrl).then(r=>r.json()).then(data=>{
      if(Array.isArray(data)&&data.length){
        skipNextSave.current=true;
        lastJsonRef.current=JSON.stringify(data);
        setBigProjects(data);
      }else{
        lastJsonRef.current=JSON.stringify(bigProjects);
      }
      setSyncStatus("synced");
    }).catch(()=>setSyncStatus("error"));
  },[cloudUrl]);
  // 資料變動時，延遲幾秒後寫回雲端
  useEffect(()=>{
    if(!cloudUrl)return;
    if(skipNextSave.current){skipNextSave.current=false;return;}
    const json=JSON.stringify(bigProjects);
    if(json===lastJsonRef.current)return;
    setSyncStatus("syncing");
    const t=setTimeout(()=>{
      fetch(cloudUrl,{method:"POST",headers:{"Content-Type":"text/plain;charset=utf-8"},body:json,mode:"no-cors"})
        .then(()=>{lastJsonRef.current=json;setSyncStatus("synced");})
        .catch(()=>setSyncStatus("error"));
    },1200);
    return()=>clearTimeout(t);
  },[bigProjects,cloudUrl]);
  // 定期向雲端拉取，同步其他人的更新
  useEffect(()=>{
    if(!cloudUrl)return;
    const iv=setInterval(()=>{
      fetch(cloudUrl).then(r=>r.json()).then(data=>{
        const json=JSON.stringify(data);
        if(Array.isArray(data)&&json!==lastJsonRef.current){
          skipNextSave.current=true;
          lastJsonRef.current=json;
          setBigProjects(data);
        }
      }).catch(()=>{});
    },15000);
    return()=>clearInterval(iv);
  },[cloudUrl]);
  const SYNC_LABEL={idle:"",loading:"⏳ 讀取中…",syncing:"🔄 同步中…",synced:"✅ 已同步",error:"⚠ 連線失敗"};
  const[drawerTaskId,setDrawerTaskId]=useState(null);
  const[drawerTab,setDrawerTab]=useState("detail");
  const[hovered,setHovered]=useState(null);
  const[holidayTip,setHolidayTip]=useState(null);
  const[editTaskId,setEditTaskId]=useState(null);
  const[editTaskName,setEditTaskName]=useState("");
  const[editBpId,setEditBpId]=useState(null);
  const[editBpName,setEditBpName]=useState("");
  const[expandedBpId,setExpandedBpId]=useState(null);
  const[newTaskModal,setNewTaskModal]=useState(null);
  const[editDateId,setEditDateId]=useState(null);
  const[mobileTab,setMobileTab]=useState("list");
  const isMobile=useWindowWidth()<768;
  const updBp=fn=>setBigProjects(bps=>bps.map(fn));
  const updTask=(tid,fn)=>setBigProjects(bps=>bps.map(bp=>({...bp,tasks:bp.tasks.map(t=>t.id===tid?fn(t):t)})));
  const findInfo=tid=>{
    for(let i=0;i<bigProjects.length;i++){const bp=bigProjects[i],t=bp.tasks.find(x=>x.id===tid);if(t)return{t,bp,color:COLORS[i%COLORS.length]};}
    return null;
  };
  const setDetailField=(tid,key,val)=>updTask(tid,t=>{
    const nd={...(t.detail||{}),[key]:val};
    if(key==="actualCost"&&!String(val).replace(/[^0-9]/g,"")){const st=JSON.parse(JSON.stringify(t.steps));st.contactDeposit.percent="";return{...t,detail:nd,steps:st};}
    return{...t,detail:nd};
  });
  const setStepField=(tid,path,val)=>updTask(tid,t=>{
    const st=JSON.parse(JSON.stringify(t.steps)),p=path.split(".");let o=st;
    for(let i=0;i<p.length-1;i++)o=o[p[i]];
    o[p[p.length-1]]=val;return{...t,steps:st};
  });
  const setBpBudget=(bpId,val)=>updBp(bp=>bp.id===bpId?{...bp,totalBudget:val?fmtN(parseN(val)):""}:bp);
  const setTaskBudget=(tid,val)=>updTask(tid,t=>({...t,taskBudget:val?fmtN(parseN(val)):""}));
  const addBigProject=()=>setBigProjects(bps=>[...bps,mkBigProject()]);
  const delBigProject=bpId=>{
    if(!window.confirm("確定要刪除此大專案及所有工程嗎？"))return;
    setBigProjects(bps=>bps.filter(bp=>bp.id!==bpId));
  };
  const addTask=bpId=>setBigProjects(bps=>bps.map(bp=>bp.id===bpId?{...bp,tasks:[...bp.tasks,mkTask()]}:bp));
  const openAddTask=bpId=>setNewTaskModal({bpId,name:"",startDate:"",endDate:""});
  const confirmAddTask=()=>{
    if(!newTaskModal)return;
    const{bpId,name,startDate,endDate}=newTaskModal;
    if(!startDate||!endDate||parseDate(endDate)<parseDate(startDate))return;
    setBigProjects(bps=>bps.map(bp=>bp.id===bpId?{...bp,tasks:[...bp.tasks,mkTask(name.trim()||"新工程",startDate,endDate)]}:bp));
    setNewTaskModal(null);
  };
  const delTask=(bpId,tid)=>{
    if(!window.confirm("確定要刪除此工程嗎？"))return;
    setBigProjects(bps=>bps.map(bp=>bp.id===bpId?{...bp,tasks:bp.tasks.filter(t=>t.id!==tid)}:bp));
    if(drawerTaskId===tid)setDrawerTaskId(null);
  };
  const toggleCollapse=bpId=>updBp(bp=>bp.id===bpId?{...bp,collapsed:!bp.collapsed}:bp);
  const toggleDone=tid=>updTask(tid,t=>({...t,done:!t.done}));
  const commitBpEdit=()=>{if(editBpName.trim())updBp(bp=>bp.id===editBpId?{...bp,name:editBpName.trim()}:bp);setEditBpId(null);};
  const commitTaskEdit=()=>{if(editTaskName.trim())updTask(editTaskId,t=>({...t,name:editTaskName.trim()}));setEditTaskId(null);};
  const allTasks=useMemo(()=>bigProjects.flatMap(bp=>bp.tasks),[bigProjects]);
  const gantt=useMemo(()=>{
    const valid=allTasks.filter(t=>t.startDate&&t.endDate);
    if(!valid.length)return{months:[],mDays:[]};
    const all=valid.flatMap(t=>[parseDate(t.startDate),parseDate(t.endDate)]);
    const mn=new Date(Math.min(...all)),mx=new Date(Math.max(...all));
    const months=[];let cur=new Date(mn.getFullYear(),mn.getMonth(),1);
    while(cur<=new Date(mx.getFullYear(),mx.getMonth(),1)){months.push(new Date(cur));cur=new Date(cur.getFullYear(),cur.getMonth()+1,1);}
    return{months,mDays:months.map(m=>daysInMonth(m.getFullYear(),m.getMonth()))};
  },[allTasks]);
  const{months,mDays}=gantt;
  const todayDate=new Date();
  const getTimePct=t=>{
    const now=new Date();now.setHours(0,0,0,0);
    const s=parseDate(t.startDate),e=parseDate(t.endDate);
    if(now<=s)return 0;if(now>=e)return 100;
    return Math.round(diffDays(s,now)/diffDays(s,e)*100);
  };
  const newTaskWorkDays=newTaskModal&&newTaskModal.startDate&&newTaskModal.endDate&&parseDate(newTaskModal.endDate)>=parseDate(newTaskModal.startDate)
    ?calcWorkDays(newTaskModal.startDate,newTaskModal.endDate):null;
  const dateInvalid=newTaskModal&&newTaskModal.startDate&&newTaskModal.endDate&&parseDate(newTaskModal.endDate)<parseDate(newTaskModal.startDate);
  const dateMissing=newTaskModal&&(!newTaskModal.startDate||!newTaskModal.endDate);
  const newTaskModalEl=newTaskModal&&(
    <div style={{position:"fixed",inset:0,background:"rgba(15,23,42,.45)",zIndex:200,display:"flex",alignItems:"center",justifyContent:"center"}}
      onClick={()=>setNewTaskModal(null)}>
      <div style={{background:"white",borderRadius:10,padding:20,width:300,maxWidth:"90vw",boxShadow:"0 10px 30px rgba(0,0,0,.2)"}} onClick={e=>e.stopPropagation()}>
        <div style={{fontSize:15,fontWeight:800,color:"#1e293b",marginBottom:14}}>＋ 新增工程</div>
        <div style={{marginBottom:10}}>
          <div style={{fontSize:11,fontWeight:700,color:"#64748b",marginBottom:4}}>工程名稱</div>
          <input autoFocus type="text" value={newTaskModal.name} placeholder="新工程"
            onChange={e=>setNewTaskModal({...newTaskModal,name:e.target.value})}
            style={{border:"1px solid #e2e8f0",borderRadius:4,padding:"6px 10px",fontSize:13,outline:"none",fontFamily:"inherit",width:"100%"}}/>
        </div>
        <div style={{display:"flex",gap:8,marginBottom:10}}>
          <div style={{flex:1}}>
            <div style={{fontSize:11,fontWeight:700,color:"#64748b",marginBottom:4}}>起始日</div>
            <input type="date" value={newTaskModal.startDate}
              onChange={e=>setNewTaskModal({...newTaskModal,startDate:e.target.value})}
              style={{border:"1px solid #e2e8f0",borderRadius:4,padding:"6px 8px",fontSize:12,outline:"none",fontFamily:"inherit",width:"100%"}}/>
          </div>
          <div style={{flex:1}}>
            <div style={{fontSize:11,fontWeight:700,color:"#64748b",marginBottom:4}}>結束日</div>
            <input type="date" value={newTaskModal.endDate}
              onChange={e=>setNewTaskModal({...newTaskModal,endDate:e.target.value})}
              style={{border:"1px solid #e2e8f0",borderRadius:4,padding:"6px 8px",fontSize:12,outline:"none",fontFamily:"inherit",width:"100%"}}/>
          </div>
        </div>
        <div style={{fontSize:12,color:dateInvalid?"#ef4444":"#334155",marginBottom:16,background:dateInvalid?"#fef2f2":"#f8fafc",border:"1px solid "+(dateInvalid?"#fecaca":"#e2e8f0"),borderRadius:6,padding:"6px 10px"}}>
          {dateInvalid?"⚠ 結束日不可早於起始日":dateMissing?"📅 請選擇起始日與結束日":`📅 工作天數：${newTaskWorkDays} 天（已扣除週末與國定假日）`}
        </div>
        <div style={{display:"flex",gap:8,justifyContent:"flex-end"}}>
          <button onClick={()=>setNewTaskModal(null)}
            style={{background:"#f1f5f9",border:"1px solid #e2e8f0",color:"#64748b",borderRadius:6,padding:"7px 16px",fontSize:12,fontWeight:700,cursor:"pointer"}}>取消</button>
          <button onClick={confirmAddTask} disabled={dateInvalid||dateMissing}
            style={{background:(dateInvalid||dateMissing)?"#cbd5e1":"#1d4ed8",border:"none",color:"white",borderRadius:6,padding:"7px 16px",fontSize:12,fontWeight:700,cursor:(dateInvalid||dateMissing)?"not-allowed":"pointer"}}>確認新增</button>
        </div>
      </div>
    </div>
  );
  const cloudSettingsEl=showCloudSettings&&(
    <div style={{position:"fixed",inset:0,background:"rgba(15,23,42,.45)",zIndex:300,display:"flex",alignItems:"center",justifyContent:"center"}}
      onClick={()=>setShowCloudSettings(false)}>
      <div style={{background:"white",borderRadius:10,padding:20,width:340,maxWidth:"90vw",boxShadow:"0 10px 30px rgba(0,0,0,.2)"}} onClick={e=>e.stopPropagation()}>
        <div style={{fontSize:15,fontWeight:800,color:"#1e293b",marginBottom:10}}>☁️ 雲端同步設定</div>
        <div style={{fontSize:12,color:"#64748b",marginBottom:10,lineHeight:1.6}}>
          貼上 Google 試算表 Apps Script 的「Web App」網址，全公司打開此網頁即可共用同一份資料（每隔約 15 秒自動同步）。留空則僅儲存在此瀏覽器。
        </div>
        <input type="text" value={cloudUrlInput} placeholder="https://script.google.com/macros/s/.../exec"
          onChange={e=>setCloudUrlInput(e.target.value)}
          style={{border:"1px solid #e2e8f0",borderRadius:4,padding:"6px 10px",fontSize:12,outline:"none",fontFamily:"inherit",width:"100%",marginBottom:14}}/>
        <div style={{display:"flex",gap:8,justifyContent:"flex-end"}}>
          <button onClick={()=>setShowCloudSettings(false)}
            style={{background:"#f1f5f9",border:"1px solid #e2e8f0",color:"#64748b",borderRadius:6,padding:"7px 16px",fontSize:12,fontWeight:700,cursor:"pointer"}}>取消</button>
          <button onClick={saveCloudUrl}
            style={{background:"#1d4ed8",border:"none",color:"white",borderRadius:6,padding:"7px 16px",fontSize:12,fontWeight:700,cursor:"pointer"}}>儲存</button>
        </div>
      </div>
    </div>
  );
  const [exporting,setExporting]=useState(false);
  const exportPDF=()=>{
    if(!months.length){alert("目前沒有任何工程資料可匯出。");return;}
    setExporting(true);
    const esc=s=>String(s??"").replace(/[&<>]/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;"}[c]));
    const TD=new Date();TD.setHours(0,0,0,0);
    const todayK=dateKey(TD);
    const dayHeaderCells=(y,mo,ndays)=>{
      let out="";
      for(let d=1;d<=ndays;d++){
        const date=new Date(y,mo,d);
        const dow=date.getDay(),isWE=dow===0||dow===6,isHD=isHoliday(date),isTD=dateKey(date)===todayK;
        const bg=isTD?"#fde68a":isHD?"#ecfdf5":isWE?"#f1f5f9":"transparent";
        const fc=isTD?"#92400e":isHD?"#15803d":isWE?"#64748b":"#475569";
        const wc=isTD?"#b45309":isHD?"#16a34a":isWE?"#94a3b8":"#94a3b8";
        const bold=(isTD||isHD)?800:500;
        out+=`<div style="width:${DAY_PX}px;flex-shrink:0;text-align:center;background:${bg};border-right:1px solid #e2e8f0;padding:3px 0;">`+
             `<div style="font-size:10px;font-weight:${bold};color:${fc};line-height:1.3;">${d}</div>`+
             `<div style="font-size:8px;color:${wc};line-height:1.2;">${WEEKDAY[dow]}</div></div>`;
      }
      return out;
    };
    const bgCells=(y,mo,ndays)=>{
      let out="";
      for(let d=1;d<=ndays;d++){
        const date=new Date(y,mo,d);
        const dow=date.getDay(),isWE=dow===0||dow===6,isHD=isHoliday(date),isTD=dateKey(date)===todayK;
        const bg=isTD?"#fef3c7":isHD?"#ecfdf5":isWE?"#f1f5f9":"transparent";
        out+=`<div style="position:absolute;left:${(d-1)*DAY_PX}px;top:0;width:${DAY_PX}px;height:100%;background:${bg};border-right:1px solid #f8fafc;"></div>`;
      }
      return out;
    };
    const pages=[];
    months.forEach((m,mi)=>{
      const y=m.getFullYear(),mo=m.getMonth(),ndays=mDays[mi];
      const mFirst=new Date(y,mo,1),mLast=new Date(y,mo+1,0);
      const W=ndays*DAY_PX;
      let ganttRows="",tableRows="";
      bigProjects.forEach((bp,bpIdx)=>{
        const bpColor=COLORS[bpIdx%COLORS.length];
        const activeTasks=bp.tasks.filter(t=>{
          if(!t.startDate||!t.endDate)return false;
          const s=parseDate(t.startDate),e=parseDate(t.endDate);
          return s<=mLast&&e>=mFirst;
        });
        if(!activeTasks.length)return;
        const bpStarts=activeTasks.map(t=>parseDate(t.startDate)),bpEnds=activeTasks.map(t=>parseDate(t.endDate));
        const bpMin=new Date(Math.min(...bpStarts)),bpMax=new Date(Math.max(...bpEnds));
        const bandS=bpMin>mFirst?bpMin:mFirst,bandE=bpMax<mLast?bpMax:mLast;
        const bandL=(bandS.getDate()-1)*DAY_PX,bandW=Math.max((diffDays(bandS,bandE)+1)*DAY_PX,DAY_PX);
        ganttRows+=`<div style="display:flex;align-items:center;height:${BP_H}px;background:${bpColor}0d;border-bottom:1px solid ${bpColor}33;">`+
          `<div style="width:${LABEL_W}px;flex-shrink:0;padding-left:10px;font-size:11px;font-weight:800;color:${bpColor};overflow:hidden;text-overflow:ellipsis;white-space:nowrap;">▼ ${esc(bp.name)}</div>`+
          `<div style="position:relative;width:${W}px;height:100%;flex-shrink:0;">${bgCells(y,mo,ndays)}`+
          `<div style="position:absolute;left:${bandL}px;top:6px;height:7px;width:${bandW}px;background:${bpColor};opacity:.28;border-radius:3px;"></div>`+
          `</div></div>`;
        tableRows+=`<tr class="bp-row"><td colspan="4">${esc(bp.name)} — 總預算 ${bp.totalBudget?esc(fmtN(parseN(bp.totalBudget))):"—"} 元</td></tr>`;
        activeTasks.forEach(t=>{
          const st=calcStatus(t),barColor=st==="待執行"?"#94a3b8":bpColor;
          const tS=parseDate(t.startDate),tE=parseDate(t.endDate);
          const clipS=tS>mFirst?tS:mFirst,clipE=tE<mLast?tE:mLast;
          const barL=(clipS.getDate()-1)*DAY_PX,barW=Math.max((diffDays(clipS,clipE)+1)*DAY_PX,DAY_PX);
          const labelColor=st==="待執行"?"#334155":"white";
          ganttRows+=`<div style="display:flex;align-items:center;height:28px;border-bottom:1px solid #f1f5f9;">`+
            `<div style="width:${LABEL_W}px;flex-shrink:0;padding-left:20px;font-size:10px;color:#475569;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;">${esc(t.name)}</div>`+
            `<div style="position:relative;width:${W}px;height:100%;flex-shrink:0;">${bgCells(y,mo,ndays)}`+
            `<div style="position:absolute;left:${barL}px;top:5px;width:${barW}px;height:18px;border-radius:9px;background:${barColor};display:flex;align-items:center;padding-left:6px;font-size:10px;font-weight:700;color:${labelColor};white-space:nowrap;overflow:hidden;">${t.done?"已完成":st}</div>`+
            `</div></div>`;
          const cls=t.done?"st-done":st==="執行中"?"st-ing":"st-wait";
          tableRows+=`<tr><td style="padding-left:16px;">${esc(t.name)}</td><td>${t.startDate} ~ ${t.endDate}</td><td>${calcWorkDays(t.startDate,t.endDate)}</td><td><span class="status-pill ${cls}">${t.done?"已完成":st}</span></td></tr>`;
        });
      });
      if(!tableRows)return;
      const isCurM=(y===TD.getFullYear()&&mo===TD.getMonth());
      const curTag=isCurM?`<span style="margin-left:8px;font-size:11px;background:#ef4444;padding:1px 7px;border-radius:8px;font-weight:700;color:white;">本月</span>`:"";
      pages.push(`<div class="pdf-page">`+
        `<div class="pdf-title-bar"><div class="pdf-title">🏗️ 工程管理系統 — 專案總覽報表</div><div class="pdf-page-tag">第 ${pages.length+1} 頁・${y}年${mo+1}月</div></div>`+
        `<div class="pdf-gantt-month-label">${y} 年 ${mo+1} 月${curTag}</div>`+
        `<div class="pdf-gantt-grid">`+
          `<div style="display:flex;border-bottom:2px solid #cbd5e1;background:#f8fafc;">`+
            `<div style="width:${LABEL_W}px;flex-shrink:0;display:flex;align-items:center;padding-left:10px;font-size:11px;font-weight:700;color:#64748b;">大專案/工程</div>`+
            dayHeaderCells(y,mo,ndays)+
          `</div>`+
          ganttRows+
        `</div>`+
        `<div class="pdf-table-title">${mo+1}月 工程清單</div>`+
        `<table class="pdf-main"><tr><th>大專案 / 工程</th><th>日期</th><th>工作天</th><th>狀態</th></tr>${tableRows}</table>`+
      `</div>`);
    });
    if(!pages.length){alert("目前沒有任何工程資料可匯出。");setExporting(false);return;}
    const PDF_CSS=`
      .pdf-root *{box-sizing:border-box;}
      .pdf-root{font-family:"PingFang TC","Microsoft JhengHei",system-ui,sans-serif;color:#1e293b;font-size:13px;}
      .pdf-page{width:1040px;padding:24px;}
      .pdf-title-bar{display:flex;justify-content:space-between;align-items:baseline;margin-bottom:14px;}
      .pdf-title{font-size:20px;font-weight:800;color:#1d4ed8;}
      .pdf-page-tag{font-size:12px;color:#94a3b8;font-weight:700;}
      .pdf-gantt-month-label{background:#1e293b;color:white;font-size:14px;font-weight:800;padding:6px 14px;border-radius:5px 5px 0 0;display:flex;align-items:center;}
      .pdf-gantt-grid{border:1px solid #e2e8f0;border-top:none;border-radius:0 0 5px 5px;margin-bottom:14px;}
      .pdf-table-title{font-size:14px;font-weight:800;color:#1e293b;margin-bottom:6px;}
      table.pdf-main{width:100%;border-collapse:collapse;font-size:12px;margin-bottom:16px;}
      table.pdf-main th{background:#f1f5f9;color:#475569;text-align:left;padding:6px 10px;font-weight:700;border:1px solid #e2e8f0;white-space:nowrap;}
      table.pdf-main td{padding:6px 10px;border:1px solid #f1f5f9;vertical-align:top;}
      table.pdf-main tr.bp-row td{background:#eff6ff;font-weight:800;color:#1d4ed8;}
      .status-pill{display:inline-block;padding:2px 10px;border-radius:10px;font-size:11px;font-weight:700;white-space:nowrap;}
      .st-done{background:#f0fdf4;color:#16a34a;border:1px solid #86efac;}
      .st-ing{background:#eff6ff;color:#2563eb;border:1px solid #93c5fd;}
      .st-wait{background:#f1f5f9;color:#64748b;border:1px solid #cbd5e1;}
    `;
    const container=document.createElement("div");
    container.style.position="fixed";container.style.left="-9999px";container.style.top="0";
    container.innerHTML=`<style>${PDF_CSS}</style><div class="pdf-root">${pages.join("")}</div>`;
    document.body.appendChild(container);
    const fname=`工程管理報表_${dateKey(new Date())}.pdf`;
    html2pdf().set({
      margin:[8,8,8,8],
      filename:fname,
      image:{type:"jpeg",quality:0.98},
      html2canvas:{scale:2,useCORS:true},
      jsPDF:{unit:"mm",format:"a4",orientation:"landscape"},
      pagebreak:{mode:["css","legacy"],after:".pdf-page"}
    }).from(container.querySelector(".pdf-root")).save().then(()=>{
      document.body.removeChild(container);setExporting(false);
    }).catch(()=>{document.body.removeChild(container);setExporting(false);});
  };
  const ganttMonthsEl=months.map((m,mi)=>{
    const days=mDays[mi];
    const isCurM=todayDate.getFullYear()===m.getFullYear()&&todayDate.getMonth()===m.getMonth();
    const todayD=isCurM?todayDate.getDate()-1:-1;
    const W=days*DAY_PX;
    const rows=[];
    bigProjects.forEach((bp,bpIdx)=>{
      const bpColor=COLORS[bpIdx%COLORS.length];
      const valid=bp.tasks.filter(t=>t.startDate&&t.endDate);
      const bpS=valid.length?new Date(Math.min(...valid.map(t=>parseDate(t.startDate)))):null;
      const bpE=valid.length?new Date(Math.max(...valid.map(t=>parseDate(t.endDate)))):null;
      rows.push({type:"bp",bp,bpColor,bpS,bpE});
      if(!bp.collapsed)bp.tasks.forEach(t=>rows.push({type:"task",t,bpColor}));
    });
    const DayCells=({total})=>Array.from({length:total},(_,d)=>{
      const cd=new Date(m.getFullYear(),m.getMonth(),d+1);
      const dow=cd.getDay(),isWE=dow===0||dow===6,isTD=d===todayD,isHD=isHoliday(cd);
      return <div key={d} style={{position:"absolute",left:d*DAY_PX,top:0,width:DAY_PX,height:"100%",background:isTD?"#fef3c7":isHD?"#ecfdf5":isWE?"#f1f5f9":"transparent",borderRight:d===total-1?"none":"1px solid #f8fafc"}}/>;
    });
    return(
      <div key={mi} style={{marginBottom:28,minWidth:LABEL_W+W}}>
        <div style={{display:"flex",alignItems:"center",background:"#1e293b",color:"white",padding:"5px 12px",borderRadius:"6px 6px 0 0",fontSize:13,fontWeight:800,letterSpacing:".05em"}}>
          {m.getFullYear()} 年 {m.getMonth()+1} 月
          {isCurM&&<span style={{marginLeft:10,fontSize:10,background:"#ef4444",padding:"1px 7px",borderRadius:8,fontWeight:700}}>本月</span>}
        </div>
        <div style={{position:"relative",border:"1px solid #e2e8f0",borderTop:"none",borderRadius:"0 0 6px 6px",overflow:"hidden"}}>
          {isCurM&&<div style={{position:"absolute",left:LABEL_W+todayD*DAY_PX+DAY_PX/2-1,top:0,bottom:0,width:2,background:"#ef4444",zIndex:20,pointerEvents:"none"}}/>}
          <div style={{display:"flex",borderBottom:"2px solid #cbd5e1",background:"#f8fafc"}}>
            <div style={{width:LABEL_W,flexShrink:0,display:"flex",alignItems:"center",paddingLeft:10,fontSize:10,fontWeight:700,color:"#64748b"}}>大專案 / 工程</div>
            {Array.from({length:days},(_,d)=>{
              const date=new Date(m.getFullYear(),m.getMonth(),d+1);
              const dow=date.getDay(),isWE=dow===0||dow===6,isHD=isHoliday(date),isTD=d===todayD;
              const bg=isTD?"#fde68a":isHD?"#ecfdf5":isWE?"#f1f5f9":"transparent";
              const fc=isTD?"#92400e":isHD?"#15803d":isWE?"#64748b":"#475569";
              const wc=isTD?"#b45309":isHD?"#16a34a":isWE?"#94a3b8":"#94a3b8";
              return(
                <div key={d} title={isHD?holidayName(date):undefined}
                  onClick={isHD?(e)=>{e.stopPropagation();const r=e.currentTarget.getBoundingClientRect();setHolidayTip({x:r.left,y:r.bottom+4,name:holidayName(date)});}:undefined}
                  style={{width:DAY_PX,flexShrink:0,textAlign:"center",borderRight:d===days-1?"none":"1px solid #e2e8f0",background:bg,paddingTop:3,paddingBottom:3,cursor:isHD?"pointer":"default"}}>
                  <div style={{fontSize:10,fontWeight:isTD||isHD?800:500,color:fc,lineHeight:1.3}}>{d+1}</div>
                  <div style={{fontSize:8,color:wc,lineHeight:1.2}}>{WEEKDAY[dow]}</div>
                </div>
              );
            })}
          </div>
          {rows.map((row,ri)=>{
            if(row.type==="bp"){
              const{bp,bpColor}=row;
              return(
                <div key={"bp-"+bp.id} style={{display:"flex",alignItems:"center",height:BP_H,background:bpColor+"0d",borderBottom:`1px solid ${bpColor}33`}}>
                  <div style={{width:LABEL_W,flexShrink:0,paddingLeft:10,paddingRight:6,fontSize:11,fontWeight:700,color:bpColor,overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>
                    {bp.collapsed?"▶ ":"▼ "}{bp.name}
                  </div>
                  <div style={{position:"relative",width:W,height:"100%",flexShrink:0}}>
                    <DayCells total={days}/>
                  </div>
                </div>
              );
            }
            const{t,bpColor}=row;
            if(!t.startDate||!t.endDate)return null;
            const st=calcStatus(t),barColor=st==="待執行"?"#94a3b8":bpColor;
            const pct=getTimePct(t),isH=hovered===t.id;
            const mFirst=new Date(m.getFullYear(),m.getMonth(),1),mLast=new Date(m.getFullYear(),m.getMonth()+1,0);
            const tS=parseDate(t.startDate),tE=parseDate(t.endDate);
            if(tS>mLast||tE<mFirst)return null;
            const clipS=tS>mFirst?tS:mFirst,clipE=tE<mLast?tE:mLast;
            const barL=(clipS.getDate()-1)*DAY_PX,barW=Math.max((diffDays(clipS,clipE)+1)*DAY_PX,DAY_PX);
            return(
              <div key={"t-"+t.id} style={{display:"flex",alignItems:"stretch",minHeight:32,borderBottom:"1px solid #f1f5f9",background:t.done?"#fafafa":"white",opacity:t.done?.7:1}}>
                <div style={{width:LABEL_W,flexShrink:0,paddingLeft:20,paddingRight:6,paddingTop:4,paddingBottom:4,display:"flex",flexDirection:"column",justifyContent:"center",gap:2,minWidth:0}}>
                  <div style={{fontSize:11,color:"#1e293b",fontWeight:700,overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>{t.name}</div>
                  <div style={{fontSize:9,color:"#94a3b8",whiteSpace:"nowrap"}}>{t.startDate.slice(5)} ~ {t.endDate.slice(5)} · {calcWorkDays(t.startDate,t.endDate)} 工作天</div>
                </div>
                <div style={{position:"relative",width:W,height:"100%",flexShrink:0}}>
                  <DayCells total={days}/>
                  <div style={{position:"absolute",left:barL,top:5,width:barW,height:22,borderRadius:11,background:barColor+"33",overflow:"visible"}}
                    onMouseEnter={()=>setHovered(t.id)} onMouseLeave={()=>setHovered(null)}>
                    {!t.done&&<div style={{position:"absolute",left:0,top:0,height:"100%",width:`${pct}%`,background:barColor,borderRadius:11}}/>}
                    {t.done&&<div style={{position:"absolute",left:0,top:"50%",width:"100%",height:2,background:"#94a3b8"}}/>}
                    {barW>44&&<span style={{position:"absolute",left:7,top:"50%",transform:"translateY(-50%)",color:t.done?"#94a3b8":"white",fontSize:10,fontWeight:700,pointerEvents:"none",textShadow:"0 1px 2px rgba(0,0,0,.4)",whiteSpace:"nowrap"}}>{t.done?"已完成":st}</span>}
                  </div>
                  {isH&&(
                    <div style={{position:"absolute",left:Math.min(barL,W-240),top:-36,background:"#1e293b",color:"white",padding:"4px 8px",borderRadius:5,fontSize:11,whiteSpace:"nowrap",zIndex:10,pointerEvents:"none"}}>
                      <strong>{t.name}</strong>
                      <span style={{color:"#94a3b8"}}> · {t.startDate} → {t.endDate}</span><br/>
                      <span style={{color:st==="執行中"?"#60a5fa":"#cbd5e1"}}>{st}</span>
                      <span style={{color:"#64748b"}}> · 時程 {pct}%</span>
                    </div>
                  )}
                </div>
              </div>
            );
          })}
        </div>
      </div>
    );
  });
  // ===== MOBILE =====
  if(isMobile){
    const drawerInfo=drawerTaskId?findInfo(drawerTaskId):null;
    return(
      <div style={{minHeight:"100vh",background:"#f0f4f8",paddingBottom:32}} onClick={()=>setHolidayTip(null)}>
        <div style={{background:"#1d4ed8",color:"white",padding:"14px 16px",display:"flex",alignItems:"center",justifyContent:"space-between",position:"sticky",top:0,zIndex:30}}>
          <div>
            <div style={{fontWeight:800,fontSize:16}}>🏗️ 工程管理系統</div>
            <div style={{color:"#93c5fd",fontSize:11,marginTop:1}}>雙擊名稱可改名</div>
          </div>
          <div style={{display:"flex",gap:6,alignItems:"center",flexShrink:0}}>
            <button onClick={()=>{setCloudUrlInput(cloudUrl);setShowCloudSettings(true);}} title="雲端同步設定"
              style={{background:"rgba(255,255,255,.2)",border:"1px solid rgba(255,255,255,.4)",color:"white",borderRadius:6,padding:"6px 9px",fontSize:12,fontWeight:700,cursor:"pointer"}}>
              ☁️
            </button>
            <button onClick={exportPDF} disabled={exporting} title="匯出PDF"
              style={{background:"rgba(255,255,255,.2)",border:"1px solid rgba(255,255,255,.4)",color:"white",borderRadius:6,padding:"6px 9px",fontSize:12,fontWeight:700,cursor:exporting?"default":"pointer",opacity:exporting?.6:1}}>
              {exporting?"匯出中...":"📄 PDF"}
            </button>
            <button onClick={addBigProject}
              style={{background:"rgba(255,255,255,.2)",border:"1px solid rgba(255,255,255,.4)",color:"white",borderRadius:6,padding:"6px 12px",fontSize:12,fontWeight:700,cursor:"pointer"}}>
              ＋ 新大專案
            </button>
          </div>
        </div>
        {SYNC_LABEL[syncStatus]&&<div style={{position:"sticky",top:54,zIndex:31,background:"#1e293b",color:"white",fontSize:12,textAlign:"center",padding:"4px"}}>{SYNC_LABEL[syncStatus]}</div>}
        <div style={{display:"flex",position:"sticky",top:SYNC_LABEL[syncStatus]?86:54,zIndex:29,background:"#f0f4f8",borderBottom:"1px solid #e2e8f0"}}>
          <button onClick={()=>setMobileTab("list")}
            style={{flex:1,padding:"9px 0",border:"none",background:"none",fontSize:13,fontWeight:700,cursor:"pointer",color:mobileTab==="list"?"#1d4ed8":"#94a3b8",borderBottom:mobileTab==="list"?"2px solid #1d4ed8":"2px solid transparent"}}>
            📋 專案清單
          </button>
          <button onClick={()=>setMobileTab("gantt")}
            style={{flex:1,padding:"9px 0",border:"none",background:"none",fontSize:13,fontWeight:700,cursor:"pointer",color:mobileTab==="gantt"?"#1d4ed8":"#94a3b8",borderBottom:mobileTab==="gantt"?"2px solid #1d4ed8":"2px solid transparent"}}>
            📊 甘特圖
          </button>
        </div>
        {mobileTab==="gantt"&&(
          <div style={{overflowX:"auto",overflowY:"hidden",padding:"10px 10px 24px",background:"white"}}>
            {ganttMonthsEl}
          </div>
        )}
        {mobileTab==="list"&&(
        <div style={{padding:"12px 12px 0"}}>
          {bigProjects.map((bp,bpIdx)=>{
            const bpColor=COLORS[bpIdx%COLORS.length];
            const total=bp.tasks.length,done=bp.tasks.filter(t=>t.done).length;
            const active=bp.tasks.filter(t=>calcStatus(t)==="執行中").length;
            const pct=total?Math.round(done/total*100):0,isExp=expandedBpId===bp.id;
            const totalBN=parseN(bp.totalBudget),usedBN=bp.tasks.reduce((s,t)=>s+parseN(t.detail?.actualCost),0);
            const over=totalBN>0&&usedBN>totalBN,pctUsed=totalBN>0?Math.min(Math.round(usedBN/totalBN*100),999):0;
            const remBN=totalBN-usedBN;
            return(
              <div key={bp.id} style={{background:"white",borderRadius:10,marginBottom:12,border:`1px solid ${bpColor}33`,overflow:"hidden",boxShadow:"0 1px 6px rgba(0,0,0,.07)"}}>
                <div style={{background:bpColor+"10",borderBottom:`1px solid ${bpColor}22`,padding:"12px 14px"}}>
                  <div style={{display:"flex",alignItems:"center",gap:6,marginBottom:8}}>
                    <span style={{color:bpColor,fontSize:19,fontWeight:900,lineHeight:1,flexShrink:0}}>▌</span>
                    {editBpId===bp.id
                      ?<input autoFocus value={editBpName} onChange={e=>setEditBpName(e.target.value)}
                          onBlur={commitBpEdit} onKeyDown={e=>{if(e.key==="Enter"||e.key==="Escape")e.currentTarget.blur();e.stopPropagation();}}
                          onClick={e=>e.stopPropagation()}
                          style={{flex:1,fontSize:15,fontWeight:700,border:"none",borderBottom:`2px solid ${bpColor}`,outline:"none",background:"transparent",fontFamily:"inherit"}}/>
                      :<span onDoubleClick={()=>{setEditBpId(bp.id);setEditBpName(bp.name);}} style={{flex:1,fontSize:15,fontWeight:700,color:"#1e293b"}}>{bp.name}</span>
                    }
                    <button onClick={()=>setExpandedBpId(id=>id===bp.id?null:bp.id)}
                      style={{background:"none",border:"none",cursor:"pointer",fontSize:15,color:bpColor,padding:"0 2px",flexShrink:0}}>{isExp?"▲":"▼"}</button>
                    <button onClick={()=>delBigProject(bp.id)}
                      style={{background:"none",border:"none",cursor:"pointer",fontSize:15,color:"#fca5a5",padding:"0 2px",flexShrink:0}}
                      onMouseEnter={e=>e.currentTarget.style.color="#ef4444"} onMouseLeave={e=>e.currentTarget.style.color="#fca5a5"}>🗑️</button>
                  </div>
                  <div style={{display:"flex",alignItems:"center",gap:6,marginBottom:totalBN>0?4:8}}>
                    <span style={{fontSize:11,color:"#64748b",flexShrink:0}}>總預算</span>
                    <input type="text" inputMode="numeric" value={bp.totalBudget} placeholder="—"
                      onChange={e=>setBpBudget(bp.id,e.target.value)} onClick={e=>e.stopPropagation()}
                      style={{border:"1px solid #e2e8f0",borderRadius:4,padding:"2px 6px",fontSize:11,outline:"none",textAlign:"right",fontFamily:"inherit",width:110}}/>
                    <span style={{fontSize:11,color:"#64748b"}}>元（含稅）</span>
                    {active>0&&<span style={{marginLeft:"auto",fontSize:11,color:"#2563eb",fontWeight:700,background:"#eff6ff",padding:"2px 7px",borderRadius:8,whiteSpace:"nowrap"}}>{active} 執行中</span>}
                  </div>
                  {totalBN>0&&(
                    <div style={{display:"flex",alignItems:"center",gap:6,marginBottom:8}}>
                      <div style={{flex:1,height:5,background:"#e2e8f0",borderRadius:3,overflow:"hidden"}}>
                        <div style={{width:`${Math.min(pctUsed,100)}%`,height:"100%",background:over?"#ef4444":bpColor,borderRadius:3}}/>
                      </div>
                      <span style={{fontSize:11,fontWeight:700,color:over?"#ef4444":bpColor,flexShrink:0}}>{pctUsed}%</span>
                      <span style={{fontSize:11,color:over?"#ef4444":"#94a3b8",flexShrink:0,whiteSpace:"nowrap"}}>
                        {over?`⚠ 超出 ${fmtN(Math.abs(remBN))}`:`剩 ${fmtN(remBN)}`}
                      </span>
                    </div>
                  )}
                  <div style={{display:"flex",alignItems:"center",gap:8}}>
                    <div style={{flex:1,height:7,background:"#e2e8f0",borderRadius:4,overflow:"hidden"}}>
                      <div style={{width:`${pct}%`,height:"100%",background:bpColor,borderRadius:4,transition:"width .3s"}}/>
                    </div>
                    <span style={{fontSize:11,color:bpColor,fontWeight:700,minWidth:32,textAlign:"right"}}>{pct}%</span>
                    <span style={{fontSize:11,color:"#94a3b8"}}>{done}/{total} 完成</span>
                  </div>
                </div>
                {isExp&&(
                  <div>
                    {bp.tasks.map(t=>{
                      const st=calcStatus(t),ss=S_STYLE[st];
                      const taskBN=parseN(t.detail?.actualCost),overTask=totalBN>0&&usedBN>totalBN&&taskBN>0;
                      const pctTask=totalBN>0&&taskBN>0?((taskBN/totalBN)*100).toFixed(1):null;
                      return(
                        <div key={t.id} style={{borderBottom:"1px solid #f1f5f9",background:t.done?"#fafafa":"white"}}>
                          <div style={{padding:"10px 14px 4px",display:"flex",alignItems:"center",gap:8}}>
                            <input type="checkbox" checked={t.done} onChange={()=>toggleDone(t.id)} style={{accentColor:bpColor,width:15,height:15,flexShrink:0,cursor:"pointer"}}/>
                            <div style={{flex:1,minWidth:0}}>
                              {editTaskId===t.id
                                ?<input autoFocus value={editTaskName} onChange={e=>setEditTaskName(e.target.value)}
                                    onBlur={commitTaskEdit} onKeyDown={e=>{if(e.key==="Enter"||e.key==="Escape")e.currentTarget.blur();e.stopPropagation();}}
                                    style={{width:"100%",fontSize:13,fontWeight:700,border:"none",borderBottom:`1.5px solid ${bpColor}`,outline:"none",background:"transparent",fontFamily:"inherit"}}/>
                                :<div onDoubleClick={()=>{setEditTaskId(t.id);setEditTaskName(t.name);}}
                                    style={{fontSize:13,fontWeight:700,color:t.done?"#94a3b8":"#1e293b",textDecoration:t.done?"line-through":"none",overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>
                                    {t.name}
                                  </div>
                              }
                              {editDateId===t.id?(
                                <div style={{display:"flex",gap:3,alignItems:"center",marginTop:2}} onClick={e=>e.stopPropagation()}>
                                  <input type="date" value={t.startDate} onChange={e=>updTask(t.id,tt=>({...tt,startDate:e.target.value}))}
                                    style={{fontSize:11,border:"1px solid #cbd5e1",borderRadius:2,padding:"1px 2px",width:108,fontFamily:"inherit"}}/>
                                  <span style={{fontSize:11,color:"#94a3b8"}}>~</span>
                                  <input type="date" value={t.endDate} onChange={e=>updTask(t.id,tt=>({...tt,endDate:e.target.value}))}
                                    style={{fontSize:11,border:"1px solid #cbd5e1",borderRadius:2,padding:"1px 2px",width:108,fontFamily:"inherit"}}/>
                                  <button onClick={()=>setEditDateId(null)} style={{background:"none",border:"none",cursor:"pointer",fontSize:14,color:bpColor,padding:0,flexShrink:0,fontWeight:700}}>✓</button>
                                </div>
                              ):(
                                <div onDoubleClick={()=>setEditDateId(t.id)} style={{fontSize:11,color:"#94a3b8",marginTop:1}}>{t.startDate?.slice(5)} → {t.endDate?.slice(5)} · {calcWorkDays(t.startDate,t.endDate)} 工作天</div>
                              )}
                            </div>
                            <span style={{background:ss.bg,color:ss.color,border:`1px solid ${ss.border}`,display:"inline-block",padding:"2px 7px",borderRadius:10,fontSize:10,fontWeight:700,whiteSpace:"nowrap",flexShrink:0}}>{st}</span>
                            <button onClick={()=>{setDrawerTaskId(id=>id===t.id?null:t.id);setDrawerTab("detail");}}
                              style={{background:bpColor+"15",border:`1px solid ${bpColor}44`,color:bpColor,borderRadius:4,padding:"4px 8px",fontSize:12,fontWeight:700,cursor:"pointer",flexShrink:0}}>📋</button>
                            <button onClick={()=>delTask(bp.id,t.id)}
                              style={{background:"none",border:"none",cursor:"pointer",color:"#fca5a5",fontSize:15,padding:0,lineHeight:1,flexShrink:0}}
                              onMouseEnter={e=>e.currentTarget.style.color="#ef4444"} onMouseLeave={e=>e.currentTarget.style.color="#fca5a5"}>🗑️</button>
                          </div>
                          <div style={{padding:"2px 14px 8px 37px",display:"flex",alignItems:"center",gap:6}}>
                            <span style={{fontSize:11,color:"#94a3b8",flexShrink:0}}>費用</span>
                            <input type="text" inputMode="numeric" value={t.detail?.actualCost||""} placeholder="—"
                              onChange={e=>{const n=String(e.target.value).replace(/[^0-9]/g,"");setDetailField(t.id,"actualCost",n?Number(n).toLocaleString("zh-TW"):"");}} onClick={e=>e.stopPropagation()}
                              style={{border:"none",borderBottom:`1px solid ${overTask?"#fca5a5":"#e2e8f0"}`,background:"transparent",fontSize:11,outline:"none",textAlign:"right",fontFamily:"inherit",width:90,color:overTask?"#ef4444":"#334155"}}/>
                            <span style={{fontSize:11,color:"#94a3b8",flexShrink:0}}>元（含稅）</span>
                            {pctTask&&<span style={{fontSize:11,fontWeight:700,color:overTask?"#ef4444":bpColor,flexShrink:0}}>{pctTask}%</span>}
                            {overTask&&<span style={{fontSize:11,color:"#ef4444"}}>⚠ 超出預算</span>}
                          </div>
                        </div>
                      );
                    })}
                    <div style={{padding:"10px 14px"}}>
                      <button onClick={()=>openAddTask(bp.id)}
                        style={{width:"100%",background:bpColor+"0d",border:`1px dashed ${bpColor}66`,color:bpColor,borderRadius:6,padding:"8px",fontSize:12,fontWeight:700,cursor:"pointer"}}>
                        ＋ 新增工程
                      </button>
                    </div>
                  </div>
                )}
              </div>
            );
          })}
        </div>
        )}
        {drawerInfo&&(
          <div style={{position:"fixed",inset:0,background:"white",zIndex:100,display:"flex",flexDirection:"column"}} onClick={e=>e.stopPropagation()}>
            <DrawerPanel taskInfo={drawerInfo} drawerTab={drawerTab} setDrawerTab={setDrawerTab} setDrawerTaskId={setDrawerTaskId} setDetailField={setDetailField} setStepField={setStepField}/>
          </div>
        )}
        {newTaskModalEl}
        {cloudSettingsEl}
      </div>
    );
  }
  // ===== DESKTOP =====
  return(
    <div style={{display:"flex",flexDirection:"column",height:"100vh",fontSize:14,overflow:"hidden"}} onClick={()=>setHolidayTip(null)}>
      {holidayTip&&(
        <div onClick={e=>e.stopPropagation()} style={{position:"fixed",left:holidayTip.x,top:holidayTip.y,background:"#fffde7",border:"1px solid #f59e0b",borderRadius:4,padding:"4px 10px",fontSize:12,color:"#92400e",fontWeight:700,boxShadow:"2px 3px 8px rgba(0,0,0,.18)",zIndex:1000,whiteSpace:"nowrap",display:"flex",alignItems:"center",gap:5}}>
          🎌 {holidayTip.name}
          <span onClick={()=>setHolidayTip(null)} style={{marginLeft:6,cursor:"pointer",color:"#d97706",fontWeight:900}}>×</span>
        </div>
      )}
      <div style={{flex:1,display:"flex",overflow:"hidden"}}>
        {/* LEFT PANEL */}
        <div style={{width:"22%",minWidth:230,display:"flex",flexDirection:"column",borderRight:"1px solid #e2e8f0",overflow:"hidden"}}>
          <div style={{background:"#1d4ed8",color:"white",padding:"10px 14px",flexShrink:0,display:"flex",alignItems:"center",justifyContent:"space-between"}}>
            <div>
              <div style={{fontWeight:"bold",fontSize:14}}>🏗️ 工程管理系統 Pro</div>
              <div style={{color:"#93c5fd",fontSize:10,marginTop:2}}>雙擊改名 · 📋 開啟詳情 · ▼▶ 折疊</div>
            </div>
            <div style={{display:"flex",gap:5,alignItems:"center",flexShrink:0}}>
              <button onClick={()=>{setCloudUrlInput(cloudUrl);setShowCloudSettings(true);}} title="雲端同步設定"
                style={{background:"rgba(255,255,255,.2)",border:"1px solid rgba(255,255,255,.4)",color:"white",borderRadius:5,padding:"4px 8px",fontSize:11,fontWeight:700,cursor:"pointer"}}>
                ☁️
              </button>
              <button onClick={addBigProject}
                style={{background:"rgba(255,255,255,.2)",border:"1px solid rgba(255,255,255,.4)",color:"white",borderRadius:5,padding:"4px 10px",fontSize:11,fontWeight:700,cursor:"pointer"}}>
                ＋ 新大專案
              </button>
            </div>
          </div>
          {SYNC_LABEL[syncStatus]&&<div style={{flexShrink:0,background:"#1e293b",color:"white",fontSize:11,textAlign:"center",padding:"3px"}}>{SYNC_LABEL[syncStatus]}</div>}
          <div style={{flex:1,overflowY:"auto",minHeight:0}}>
            {bigProjects.map((bp,bpIdx)=>{
              const bpColor=COLORS[bpIdx%COLORS.length];
              const done=bp.tasks.filter(t=>t.done).length;
              const active=bp.tasks.filter(t=>calcStatus(t)==="執行中").length;
              const totalBN=parseN(bp.totalBudget),usedBN=bp.tasks.reduce((s,t)=>s+parseN(t.detail?.actualCost),0);
              const over=totalBN>0&&usedBN>totalBN,pctUsed=totalBN>0?Math.min(Math.round(usedBN/totalBN*100),999):0;
              const remBN=totalBN-usedBN;
              return(
                <React.Fragment key={bp.id}>
                  <div style={{display:"flex",alignItems:"center",gap:5,padding:"4px 8px",minHeight:BP_H,background:bpColor+"15",borderBottom:`1px solid ${bpColor}33`,borderLeft:`3px solid ${bpColor}`}}>
                    <button onClick={()=>toggleCollapse(bp.id)}
                      style={{background:"none",border:"none",cursor:"pointer",color:bpColor,fontSize:11,padding:0,flexShrink:0,width:16,textAlign:"center"}}>
                      {bp.collapsed?"▶":"▼"}
                    </button>
                    {editBpId===bp.id
                      ?<input autoFocus value={editBpName} onChange={e=>setEditBpName(e.target.value)}
                          onBlur={commitBpEdit} onKeyDown={e=>{if(e.key==="Enter"||e.key==="Escape")e.currentTarget.blur();e.stopPropagation();}}
                          onClick={e=>e.stopPropagation()}
                          style={{flex:1,fontSize:12,fontWeight:700,border:"none",borderBottom:`2px solid ${bpColor}`,outline:"none",background:"transparent",fontFamily:"inherit",minWidth:0}}/>
                      :<span onDoubleClick={()=>{setEditBpId(bp.id);setEditBpName(bp.name);}} title="雙擊改名"
                          style={{flex:1,fontSize:12,fontWeight:700,color:bpColor,cursor:"pointer",overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>
                          {bp.name}
                        </span>
                    }
                    <span style={{fontSize:10,color:bpColor+"99",whiteSpace:"nowrap",flexShrink:0}}>{done}/{bp.tasks.length}</span>
                    {active>0&&<span style={{fontSize:10,color:"#2563eb",background:"#eff6ff",padding:"1px 5px",borderRadius:6,fontWeight:700,flexShrink:0,whiteSpace:"nowrap"}}>{active}執行</span>}
                    {over&&<span style={{fontSize:10,color:"#ef4444",background:"#fff1f2",padding:"1px 5px",borderRadius:6,fontWeight:700,flexShrink:0}}>⚠超</span>}
                    <button onClick={e=>{e.stopPropagation();openAddTask(bp.id);}}
                      style={{background:"none",border:"none",cursor:"pointer",color:bpColor,fontSize:15,padding:"0 1px",lineHeight:1,flexShrink:0}}>＋</button>
                    <button onClick={e=>{e.stopPropagation();delBigProject(bp.id);}}
                      style={{background:"none",border:"none",cursor:"pointer",color:"#fca5a5",fontSize:13,padding:"0 1px",lineHeight:1,flexShrink:0}}
                      onMouseEnter={e=>e.currentTarget.style.color="#ef4444"} onMouseLeave={e=>e.currentTarget.style.color="#fca5a5"}>🗑️</button>
                  </div>
                  {!bp.collapsed&&(
                    <div style={{background:over?"#fff1f2":bpColor+"08",borderBottom:`1px solid ${over?"#fecaca":bpColor+"18"}`,padding:"4px 8px 4px 24px"}}>
                      <div style={{display:"flex",alignItems:"center",gap:5,marginBottom:totalBN>0?3:0}}>
                        <span style={{fontSize:10,color:"#94a3b8",flexShrink:0}}>💰 總預算</span>
                        <input type="text" inputMode="numeric" value={bp.totalBudget} placeholder="—"
                          onChange={e=>setBpBudget(bp.id,e.target.value)} onClick={e=>e.stopPropagation()}
                          style={{border:"none",background:"transparent",fontSize:10,outline:"none",textAlign:"right",fontFamily:"inherit",flex:1,minWidth:0,color:"#334155"}}/>
                        <span style={{fontSize:10,color:"#94a3b8",flexShrink:0}}>元（含稅）</span>
                      </div>
                      {totalBN>0&&(
                        <div style={{display:"flex",alignItems:"center",gap:5}}>
                          <div style={{flex:1,height:4,background:"#e2e8f0",borderRadius:2,overflow:"hidden"}}>
                            <div style={{width:`${Math.min(pctUsed,100)}%`,height:"100%",background:over?"#ef4444":bpColor,borderRadius:2,transition:"width .3s"}}/>
                          </div>
                          <span style={{fontSize:10,fontWeight:700,color:over?"#ef4444":bpColor,flexShrink:0,minWidth:28,textAlign:"right"}}>{pctUsed}%</span>
                          <span style={{fontSize:10,color:over?"#ef4444":"#64748b",flexShrink:0,whiteSpace:"nowrap"}}>
                            {over?`⚠ 超出 ${fmtN(Math.abs(remBN))}`:`剩 ${fmtN(remBN)}`}
                          </span>
                        </div>
                      )}
                    </div>
                  )}
                  {!bp.collapsed&&bp.tasks.map(t=>{
                    const st=calcStatus(t),ss=S_STYLE[st];
                    const taskBN=parseN(t.detail?.actualCost),overTask=totalBN>0&&usedBN>totalBN&&taskBN>0;
                    const pctTask=totalBN>0&&taskBN>0?((taskBN/totalBN)*100).toFixed(1):null;
                    return(
                      <div key={t.id} className="t-row" style={{borderBottom:"1px solid #f1f5f9",background:"white",opacity:t.done?.65:1}}>
                        <div style={{display:"flex",alignItems:"center",gap:4,padding:"5px 5px 1px 20px"}}>
                          <span style={{color:bpColor,fontSize:12,flexShrink:0,lineHeight:1}}>·</span>
                          {editTaskId===t.id
                            ?<input autoFocus value={editTaskName} onChange={e=>setEditTaskName(e.target.value)}
                                onBlur={commitTaskEdit} onKeyDown={e=>{if(e.key==="Enter")commitTaskEdit();if(e.key==="Escape")setEditTaskId(null);e.stopPropagation();}}
                                onClick={e=>e.stopPropagation()}
                                style={{flex:1,fontSize:11,fontWeight:700,border:"none",borderBottom:`1.5px solid ${bpColor}`,outline:"none",background:"transparent",fontFamily:"inherit",minWidth:0}}/>
                            :<span onDoubleClick={()=>{setEditTaskId(t.id);setEditTaskName(t.name);}} title="雙擊改名"
                                style={{flex:1,fontSize:11,fontWeight:700,cursor:"pointer",overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap",textDecoration:t.done?"line-through":"none",color:t.done?"#94a3b8":"#1e293b"}}>
                                {t.name}
                              </span>
                          }
                          <span style={{...ss,display:"inline-block",padding:"1px 5px",borderRadius:8,fontSize:10,fontWeight:700,border:"1px solid",whiteSpace:"nowrap",flexShrink:0}}>{st}</span>
                          <button onClick={e=>{e.stopPropagation();setDrawerTaskId(id=>id===t.id?null:t.id);setDrawerTab("detail");}}
                            style={{background:"none",border:"none",cursor:"pointer",color:"#bfdbfe",fontSize:12,padding:"0 1px",lineHeight:1,flexShrink:0}}
                            onMouseEnter={e=>e.currentTarget.style.color=bpColor} onMouseLeave={e=>e.currentTarget.style.color="#bfdbfe"}>📋</button>
                          <button onClick={e=>{e.stopPropagation();toggleDone(t.id);}}
                            style={{background:"none",border:"none",cursor:"pointer",padding:"0 1px",lineHeight:1,flexShrink:0}}>
                            <input type="checkbox" checked={t.done} readOnly style={{width:12,height:12,cursor:"pointer",accentColor:bpColor,pointerEvents:"none"}}/>
                          </button>
                          <button onClick={e=>{e.stopPropagation();delTask(bp.id,t.id);}}
                            style={{background:"none",border:"none",cursor:"pointer",color:"#fca5a5",fontSize:12,padding:"0 1px",lineHeight:1,flexShrink:0}}
                            onMouseEnter={e=>e.currentTarget.style.color="#ef4444"} onMouseLeave={e=>e.currentTarget.style.color="#fca5a5"}>🗑️</button>
                        </div>
                        <div style={{display:"flex",alignItems:"center",gap:4,padding:"1px 5px 5px 28px"}}>
                          <span style={{fontSize:10,color:"#94a3b8",flexShrink:0}}>費用</span>
                          <input type="text" inputMode="numeric" value={t.detail?.actualCost||""} placeholder="—"
                            onChange={e=>{const n=String(e.target.value).replace(/[^0-9]/g,"");setDetailField(t.id,"actualCost",n?Number(n).toLocaleString("zh-TW"):"");}} onClick={e=>e.stopPropagation()}
                            style={{border:"none",borderBottom:`1px solid ${overTask?"#fca5a5":"#e2e8f0"}`,background:"transparent",fontSize:10,outline:"none",textAlign:"right",fontFamily:"inherit",width:80,color:overTask?"#ef4444":"#334155"}}/>
                          <span style={{fontSize:10,color:"#94a3b8",flexShrink:0}}>元（含稅）</span>
                          {pctTask&&<span style={{fontSize:10,fontWeight:700,color:overTask?"#ef4444":bpColor,marginLeft:2,flexShrink:0}}>{pctTask}%</span>}
                          {overTask&&<span style={{fontSize:10,color:"#ef4444",flexShrink:0}}>⚠</span>}
                        </div>
                        <div style={{padding:"0 5px 6px 28px"}}>
                          {editDateId===t.id?(
                            <div style={{display:"flex",gap:3,alignItems:"center"}} onClick={e=>e.stopPropagation()}>
                              <input type="date" value={t.startDate} onChange={e=>updTask(t.id,tt=>({...tt,startDate:e.target.value}))}
                                style={{fontSize:10,border:"1px solid #cbd5e1",borderRadius:2,padding:"1px 2px",width:104,fontFamily:"inherit"}}/>
                              <span style={{fontSize:10,color:"#94a3b8"}}>~</span>
                              <input type="date" value={t.endDate} onChange={e=>updTask(t.id,tt=>({...tt,endDate:e.target.value}))}
                                style={{fontSize:10,border:"1px solid #cbd5e1",borderRadius:2,padding:"1px 2px",width:104,fontFamily:"inherit"}}/>
                              <button onClick={()=>setEditDateId(null)} style={{background:"none",border:"none",cursor:"pointer",fontSize:12,color:bpColor,padding:0,flexShrink:0,fontWeight:700}}>✓</button>
                            </div>
                          ):(
                            <div onDoubleClick={()=>setEditDateId(t.id)} title="雙擊修改日期" style={{fontSize:10,color:"#94a3b8",cursor:"pointer",whiteSpace:"nowrap"}}>
                              📅 {t.startDate} ~ {t.endDate} · {calcWorkDays(t.startDate,t.endDate)} 工作天
                            </div>
                          )}
                        </div>
                      </div>
                    );
                  })}
                </React.Fragment>
              );
            })}
          </div>
        </div>
        {/* GANTT PANEL */}
        <div style={{flex:1,display:"flex",flexDirection:"column",overflow:"hidden",position:"relative"}}>
          <div style={{background:"#0f172a",color:"white",padding:"10px 16px",flexShrink:0,display:"flex",alignItems:"center",gap:10}}>
            <div>
              <div style={{fontWeight:"bold",fontSize:14}}>📊 甘特圖 — 大專案群組視圖</div>
              <div style={{color:"#475569",fontSize:10,marginTop:1}}>紅線=今日 · 彩色標題=大專案 · Hover 查看詳情</div>
            </div>
            <div style={{marginLeft:"auto",display:"flex",gap:10,alignItems:"center"}}>
              <span style={{fontSize:10,color:"#6b7280"}}>週末</span>
              <span style={{display:"inline-block",width:12,height:12,background:"#f1f5f9",border:"1px solid #cbd5e1",borderRadius:2}}/>
              <span style={{fontSize:10,color:"#6b7280",marginLeft:4}}>國定假日</span>
              <span style={{display:"inline-block",width:12,height:12,background:"#ecfdf5",border:"1px solid #a7f3d0",borderRadius:2}}/>
              <button onClick={exportPDF} disabled={exporting}
                style={{marginLeft:8,background:exporting?"#475569":"#2563eb",border:"none",color:"white",borderRadius:5,padding:"5px 12px",fontSize:12,fontWeight:700,cursor:exporting?"default":"pointer"}}>
                {exporting?"匯出中...":"📄 匯出PDF"}
              </button>
            </div>
          </div>
          <div style={{flex:1,overflowY:"auto",overflowX:"auto",padding:"14px 16px 24px",background:"white"}}>
            {ganttMonthsEl}
          </div>
          {drawerTaskId&&(()=>{
            const info=findInfo(drawerTaskId);if(!info)return null;
            return(
              <div style={{position:"absolute",top:0,right:0,bottom:0,width:360,background:"white",borderLeft:`2px solid ${info.color}`,display:"flex",flexDirection:"column",zIndex:50,boxShadow:"-4px 0 16px rgba(0,0,0,.08)"}}>
                <DrawerPanel taskInfo={info} drawerTab={drawerTab} setDrawerTab={setDrawerTab} setDrawerTaskId={setDrawerTaskId} setDetailField={setDetailField} setStepField={setStepField}/>
              </div>
            );
          })()}
        </div>
      </div>
      {newTaskModalEl}
      {cloudSettingsEl}
    </div>
  );
}
ReactDOM.createRoot(document.getElementById("root")).render(<App/>);
</script>
</body>
</html>
