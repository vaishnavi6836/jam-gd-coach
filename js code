const $=id=>document.getElementById(id);
let sample=null,dict=null;
(async()=>{try{sample=await claude.use("sample")}catch(e){}
try{dict=await claude.use("dictation")}catch(e){}
if(dict){$("jmic").classList.remove("hide");$("gmic").classList.remove("hide")}
if(!sample){$("jeval").disabled=true;$("jeval").title="AI is unavailable in this view"}})();

document.querySelectorAll("#nav button").forEach(b=>b.onclick=()=>{
document.querySelectorAll("#nav button").forEach(x=>x.classList.toggle("on",x===b));
["jam","gd","guide"].forEach(s=>$(s).classList.toggle("hide",s!==b.dataset.t))});

const esc=s=>String(s).replace(/[&<>]/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;"}[c]));
async function ask(prompt){
 if(!sample) throw new Error("AI isn’t available on this page view.");
 return await sample.json(prompt);
}
function mic(btn,ta){
 let on=false;
 btn.onclick=async()=>{
  if(!dict)return;
  try{
   if(on){await dict.stop();return}
   dict.onFinal(({text})=>{ta.value+=(ta.value&&!ta.value.endsWith(" ")?" ":"")+text;ta.dispatchEvent(new Event("input"))});
   dict.onState(({state})=>{on=state==="listening";btn.textContent=on?"⏹ Stop dictation":"🎤 Dictate"});
   await dict.start();
  }catch(e){btn.textContent="🎤 Dictation unavailable"}
 };
}
mic($("jmic"),$("jtext"));mic($("gmic"),$("gtext"));

/* ---------- JAM ---------- */
let jt=null,jiv=null,left=60;
function setTimer(){$("jtime").textContent=left;$("jbar").style.width=((60-left)/60*100)+"%"}
function jstop(){clearInterval(jiv);jiv=null;$("jstart").classList.remove("hide");$("jstop").classList.add("hide");$("jstart").textContent="Restart"}
$("jnew").onclick=async()=>{
 $("jnew").disabled=true;$("jtopic").textContent="Thinking of a topic…";
 try{
  const r=await ask(`Give ONE fresh Just A Minute (JAM) speaking topic for Indian campus/MBA practice. Category: ${$("jcat").value}. Short (2-8 words). Return JSON: {"topic":"..."}`);
  jt=r.topic;$("jtopic").textContent=jt;$("jstart").disabled=false;
 }catch(e){$("jtopic").textContent="Couldn’t fetch a topic. Try again."}
 $("jnew").disabled=false;
};
$("jstart").onclick=()=>{
 left=60;setTimer();$("jtext").value="";$("jfb").classList.add("hide");
 $("jstart").classList.add("hide");$("jstop").classList.remove("hide");
 jiv=setInterval(()=>{left--;setTimer();if(left<=0)jstop()},1000);
};
$("jstop").onclick=jstop;
$("jtext").oninput=()=>{const n=$("jtext").value.trim().split(/\s+/).filter(Boolean).length;$("jwc").textContent=n?n+" words":""};
$("jeval").onclick=async()=>{
 const t=$("jtext").value.trim();
 if(!jt||t.split(/\s+/).length<8){alert("Pick a topic and speak or type at least a few sentences first.");return}
 const used=60-left||60;
 $("jeval").disabled=true;$("jfb").classList.remove("hide");$("jfb").innerHTML="<p class='mute'>Evaluating…</p>";
 try{
  const r=await ask(`You are a strict but kind JAM (Just A Minute) coach. Topic: "${jt}". The speaker talked for about ${used} seconds. Transcript (may be dictated, so ignore punctuation errors): """${t}""". Evaluate on a 1-10 scale. Return JSON: {"fluency":n,"content":n,"structure":n,"vocabulary":n,"overall":n,"repetitions":["..."],"fillers":["..."],"strengths":["..."],"improvements":["..."],"sample_opening":"a stronger 2-sentence opening for this topic"}`);
  const sc=["fluency","content","structure","vocabulary"].map(k=>`<div><b>${r[k]}/10</b>${k}</div>`).join("");
  const li=a=>"<ul>"+(a||[]).map(x=>`<li>${esc(x)}</li>`).join("")+"</ul>";
  $("jfb").innerHTML=`<h3 style="margin-top:0">Overall: ${r.overall}/10</h3><div class="score">${sc}</div>
  <h3>What worked</h3>${li(r.strengths)}<h3>Improve next time</h3>${li(r.improvements)}
  <h3>Repetition</h3>${li(r.repetitions)}<h3>Fillers</h3>${li(r.fillers)}
  <h3>A stronger opening</h3><p>${esc(r.sample_opening)}</p>`;
 }catch(e){$("jfb").innerHTML="<p>Couldn’t evaluate right now. Please try again.</p>"}
 $("jeval").disabled=false;
};

/* ---------- GD ---------- */
let gt=null,log=[];
function render(){
 $("chat").innerHTML=log.map(m=>`<div class="msg ${m.who==="You"?"me":""}"><small>${esc(m.who)}</small>${esc(m.text)}</div>`).join("");
 $("chat").scrollTop=1e9;
}
$("gnew").onclick=async()=>{
 $("gnew").disabled=true;$("gtopic").textContent="Thinking of a topic…";log=[];render();$("gfb").classList.add("hide");
 try{
  const r=await ask(`Give ONE Group Discussion topic for Indian MBA/placement practice. Category: ${$("gcat").value}. Make it debatable. Return JSON: {"topic":"..."}`);
  gt=r.topic;$("gtopic").textContent=gt;
  ["gtext","gsend","gend"].forEach(i=>$(i).disabled=false);
  log.push({who:"Moderator",text:"The floor is open. Please start the discussion."});render();
 }catch(e){$("gtopic").textContent="Couldn’t fetch a topic. Try again."}
 $("gnew").disabled=false;
};
$("gsend").onclick=async()=>{
 const t=$("gtext").value.trim();if(!t)return;
 log.push({who:"You",text:t});$("gtext").value="";render();
 $("gsend").disabled=true;
 const hist=log.map(m=>m.who+": "+m.text).join("\n");
 try{
  const r=await ask(`Simulate a realistic Group Discussion on "${gt}". Participants: Arjun (aggressive, interrupts, strong opinions), Meera (quiet analyst, speaks rarely but with sharp, data-backed insight), Karan (moderate, builds consensus, cites examples). Conversation so far:\n${hist}\nWrite the next 2 or 3 short interventions (max 45 words each) reacting to the latest point by "You" and each other. Not every participant must speak. Return JSON array: [{"who":"Arjun","text":"..."}]`);
  (Array.isArray(r)?r:[]).forEach(m=>log.push({who:m.who,text:m.text}));
 }catch(e){log.push({who:"Moderator",text:"Technical hiccup. Please continue."})}
 render();$("gsend").disabled=false;
};
$("gend").onclick=async()=>{
 const mine=log.filter(m=>m.who==="You");
 if(!mine.length){alert("Make at least one point first.");return}
 $("gend").disabled=true;$("gfb").classList.remove("hide");$("gfb").innerHTML="<p class='mute'>Evaluating…</p>";
 try{
  const r=await ask(`You are an expert GD evaluator. Topic: "${gt}". Full discussion:\n${log.map(m=>m.who+": "+m.text).join("\n")}\nEvaluate only "You". Score 1-10. Return JSON: {"initiative":n,"content":n,"communication":n,"teamwork":n,"overall":n,"strengths":["..."],"improvements":["..."],"missed_angles":["angles or facts they could have used"],"one_line_verdict":"..."}`);
  const sc=["initiative","content","communication","teamwork"].map(k=>`<div><b>${r[k]}/10</b>${k}</div>`).join("");
  const li=a=>"<ul>"+(a||[]).map(x=>`<li>${esc(x)}</li>`).join("")+"</ul>";
  $("gfb").innerHTML=`<h3 style="margin-top:0">Overall: ${r.overall}/10</h3><p>${esc(r.one_line_verdict)}</p><div class="score">${sc}</div>
  <h3>What worked</h3>${li(r.strengths)}<h3>Improve next time</h3>${li(r.improvements)}<h3>Angles you missed</h3>${li(r.missed_angles)}`;
 }catch(e){$("gfb").innerHTML="<p>Couldn’t evaluate right now. Please try again.</p>"}
 $("gend").disabled=false;
};
