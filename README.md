<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Rialo Knowledge Quiz</title>

<style>
:root{
  --primary:#6c63ff;
  --secondary:#111827;
  --success:#22c55e;
  --danger:#ef4444;
  --bg:#0f172a;
  --card:#111827;
  --muted:#94a3b8;
}

*{box-sizing:border-box;margin:0;padding:0}

body{
  font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto;
  background:radial-gradient(circle at top,#1e293b,#020617);
  color:#e5e7eb;
  min-height:100vh;
  display:flex;
  justify-content:center;
  align-items:center;
}

.app{
  width:100%;
  max-width:820px;
  background:linear-gradient(180deg,#020617,#020617cc);
  border-radius:20px;
  box-shadow:0 30px 80px rgba(0,0,0,.7);
  overflow:hidden;
}

.header{
  padding:28px;
  border-bottom:1px solid #1e293b;
}
.logo{
  font-size:28px;
  font-weight:800;
  letter-spacing:.4px;
}
.logo span{color:var(--primary)}
.sub{
  color:var(--muted);
  margin-top:6px;
}

.screen{display:none;padding:32px}
.screen.active{display:block;animation:fade .4s ease}

@keyframes fade{
  from{opacity:0;transform:translateY(8px)}
  to{opacity:1;transform:none}
}

.start{
  text-align:center;
}
.start h2{font-size:32px}
.start p{
  color:var(--muted);
  margin:18px 0 32px;
  line-height:1.6;
}

.btn{
  padding:14px 34px;
  border-radius:999px;
  border:none;
  cursor:pointer;
  font-size:16px;
  font-weight:600;
  background:linear-gradient(135deg,var(--primary),#8b5cf6);
  color:white;
  transition:.25s ease;
}
.btn:hover{transform:translateY(-1px);box-shadow:0 10px 30px #6c63ff55}
.btn:disabled{opacity:.4;cursor:not-allowed}

.quiz-top{
  display:flex;
  justify-content:space-between;
  align-items:center;
  margin-bottom:18px;
}
.progress{
  height:6px;
  background:#1e293b;
  border-radius:6px;
  overflow:hidden;
}
.progress span{
  display:block;
  height:100%;
  width:0%;
  background:linear-gradient(90deg,var(--primary),#8b5cf6);
  transition:.4s ease;
}

.timer{
  font-size:14px;
  color:var(--muted);
}

.question{
  margin:26px 0;
  font-size:22px;
  line-height:1.5;
}

.options{
  display:grid;
  gap:14px;
}

.option{
  padding:18px;
  border-radius:14px;
  background:#020617;
  border:1px solid #1e293b;
  cursor:pointer;
  transition:.25s ease;
}
.option:hover{border-color:var(--primary)}
.option.correct{border-color:var(--success);background:#052e1a}
.option.wrong{border-color:var(--danger);background:#2a0a0a}

.feedback{
  margin-top:18px;
  font-weight:600;
}

.results{
  text-align:center;
}
.score{
  font-size:64px;
  font-weight:800;
  margin:24px 0;
}
.tag{
  color:var(--muted);
  margin-bottom:30px;
}

.footer{
  padding:20px;
  border-top:1px solid #1e293b;
  text-align:center;
  font-size:13px;
  color:#64748b;
}
</style>
</head>

<body>
<div class="app">

  <div class="header">
    <div class="logo">RIALO<span>.quiz</span></div>
    <div class="sub">Protocol Knowledge Assessment</div>
  </div>

  <!-- START -->
  <section id="start" class="screen start active">
    <h2>Test your understanding</h2>
    <p>
      A carefully designed knowledge check based on Rialo’s architecture,
      execution model and design philosophy.
    </p>
    <button class="btn" onclick="startQuiz()">Start Assessment</button>
  </section>

  <!-- QUIZ -->
  <section id="quiz" class="screen">
    <div class="quiz-top">
      <div class="timer">Question <span id="qNo"></span>/<span id="total"></span></div>
      <div class="timer">⏱ <span id="time">20</span>s</div>
    </div>
    <div class="progress"><span id="bar"></span></div>

    <div class="question" id="question"></div>
    <div class="options" id="options"></div>
    <div class="feedback" id="feedback"></div>

    <div style="margin-top:30px">
      <button id="next" class="btn" onclick="next()" disabled>Next</button>
    </div>
  </section>

  <!-- RESULT -->
  <section id="result" class="screen results">
    <h2>Assessment Complete</h2>
    <div class="score" id="score"></div>
    <div class="tag" id="remark"></div>
    <button class="btn" onclick="restart()">Retake Quiz</button>
  </section>

  <div class="footer">
    © 2026 Rialo — Educational Interface Demo
  </div>
</div>

<script>
const DATA=[
{q:"What philosophical framework does Rialo adopt?",o:["Monolithic","Modularity","Supermodularity","Microservices"],a:2},
{q:"Supermodularity refers to:",o:["Independent systems","Zero overhead","Emergent integration value","Strict isolation"],a:2},
{q:"Integration is treated as:",o:["Ideology","Economic decision","Hack","Risk"],a:1},
{q:"Execution ISA used by Rialo?",o:["ARM","x86","WASM","RISC-V"],a:3}
];

let i=0,score=0,time=20,timer;

const $=id=>document.getElementById(id);

function startQuiz(){
  $("start").classList.remove("active");
  $("quiz").classList.add("active");
  $("total").textContent=DATA.length;
  load();
}

function load(){
  clearInterval(timer);
  time=20;
  $("time").textContent=time;
  $("feedback").textContent="";
  $("next").disabled=true;
  $("options").innerHTML="";

  const d=DATA[i];
  $("qNo").textContent=i+1;
  $("question").textContent=d.q;
  $("bar").style.width=((i)/DATA.length)*100+"%";

  d.o.forEach((t,idx)=>{
    const div=document.createElement("div");
    div.className="option";
    div.textContent=t;
    div.onclick=()=>select(idx,div);
    $("options").appendChild(div);
  });

  timer=setInterval(()=>{
    time--;
    $("time").textContent=time;
    if(time===0){clearInterval(timer);autoFail()}
  },1000);
}

function select(idx,el){
  clearInterval(timer);
  const d=DATA[i];
  [...document.querySelectorAll(".option")].forEach((o,j)=>{
    o.onclick=null;
    if(j===d.a) o.classList.add("correct");
    if(j===idx && idx!==d.a) o.classList.add("wrong");
  });
  if(idx===d.a){score++;$("feedback").textContent="Correct"} 
  else {$("feedback").textContent="Incorrect"}
  $("next").disabled=false;
}

function autoFail(){
  select(-1, null);
}

function next(){
  i++;
  if(i<DATA.length) load();
  else showResult();
}

function showResult(){
  $("quiz").classList.remove("active");
  $("result").classList.add("active");
  const pct=Math.round(score/DATA.length*100);
  $("score").textContent=pct+"%";
  $("remark").textContent=
    pct>=80?"Excellent understanding":
    pct>=60?"Good conceptual clarity":
    "Needs more exploration";
}

function restart(){
  i=0;score=0;
  $("result").classList.remove("active");
  $("start").classList.add("active");
}
</script>
</body>
</html>
