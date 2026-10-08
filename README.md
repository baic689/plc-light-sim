<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>PLC仿真：8路彩灯 + 段码显示</title>
<style>
*{box-sizing:border-box;font-family:"Microsoft Yahei",sans-serif}
body{background:#1a1a2e;color:#fff;padding:24px;max-width:720px;margin:0 auto}
.box{background:#27293d;padding:20px;border-radius:12px;margin-bottom:16px}
.light-row{display:flex;gap:10px;margin:12px 0;flex-wrap:wrap}
.light{width:42px;height:42px;border-radius:50%;background:#333;border:2px solid #555;transition:0.2s}
.light.on{background:#ffea00;box-shadow:0 0 18px #ffea00}
.key-row{display:flex;gap:8px;flex-wrap:wrap;margin:12px 0}
.key{width:50px;height:50px;background:#446;border:none;color:#fff;font-size:20px;border-radius:8px;cursor:pointer}
.key:active{background:#668}
.seg-display{position:relative;width:100px;height:160px;margin:10px;background:#111;border-radius:8px}
.seg{position:absolute;background:#333;transition:0.2s}
.seg.on{background:#f22;box-shadow:0 0 12px #f22}
.seg-a{top:8px;left:22px;width:56px;height:8px;border-radius:4px}
.seg-b{top:18px;right:8px;width:8px;height:56px;border-radius:4px}
.seg-c{bottom:18px;right:8px;width:8px;height:56px;border-radius:4px}
.seg-d{bottom:8px;left:22px;width:56px;height:8px;border-radius:4px}
.seg-e{bottom:18px;left:8px;width:8px;height:56px;border-radius:4px}
.seg-f{top:18px;left:8px;width:8px;height:56px;border-radius:4px}
.seg-g{top:76px;left:22px;width:56px;height:8px;border-radius:4px}
.info{color:#acc;font-size:14px;line-height:1.6}
</style>
</head>
<body>
<h2>PLC逻辑仿真｜8路彩灯 + 七段数码管</h2>
<div class="box">
  <div class="info">功能说明：按下数字1‑8 → 对应编号彩灯点亮，段码屏同步显示该数字；每次只保留最新一组输出</div>
  <div>彩灯 L1‑L8：</div>
  <div class="light-row" id="lights"></div>
  <div style="margin-top:16px">七段数码管输出：</div>
  <div class="seg-display">
    <div class="seg seg-a" data-seg="a"></div>
    <div class="seg seg-b" data-seg="b"></div>
    <div class="seg seg-c" data-seg="c"></div>
    <div class="seg seg-d" data-seg="d"></div>
    <div class="seg seg-e" data-seg="e"></div>
    <div class="seg seg-f" data-seg="f"></div>
    <div class="seg seg-g" data-seg="g"></div>
  </div>
  <div style="margin-top:16px">输入按钮(I0.0‑I0.7 对应数字1‑8)：</div>
  <div class="key-row" id="keys"></div>
  <div class="info" style="margin-top:12px">当前激活输入：<span id="cur">无</span></div>
</div>

<script>
// PLC IO映射
// 输入：I0.0=1 , I0.1=2 … I0.7=8
// 输出彩灯：Q0.0=L1 , Q0.1=L2 … Q0.7=L8
// 七段真值表 a‑g：1亮 0灭
const segMap = {
  0: [1,1,1,1,1,1,0],
  1: [0,1,1,0,0,0,0],
  2: [1,1,0,1,1,0,1],
  3: [1,1,1,1,0,0,1],
  4: [0,1,1,0,0,1,1],
  5: [1,0,1,1,0,1,1],
  6: [1,0,1,1,1,1,1],
  7: [1,1,1,0,0,0,0],
  8: [1,1,1,1,1,1,1],
  9: [1,1,1,1,0,1,1],
  null:[0,0,0,0,0,0,0]
};

const lightWrap = document.getElementById('lights');
const keyWrap = document.getElementById('keys');
const curText = document.getElementById('cur');

// 生成8个彩灯
let lightArr = [];
for(let i=1;i<=8;i++){
  let d = document.createElement('div');
  d.className = 'light';
  d.title = 'L'+i;
  lightWrap.appendChild(d);
  lightArr.push(d);
}

// 生成1‑8按钮
for(let n=1;n<=8;n++){
  let btn = document.createElement('button');
  btn.className = 'key';
  btn.textContent = n;
  btn.onclick = ()=> plcTask(n);
  keyWrap.appendChild(btn);
}

// 更新数码管
function setSeg(num){
  let arr = segMap[num] || segMap[null];
  document.querySelectorAll('.seg').forEach((el,idx)=>{
    el.classList.toggle('on', arr[idx]===1);
  })
}

// PLC控制逻辑：先全部复位，再点亮对应输出
function plcTask(n){
  lightArr.forEach(led=>led.classList.remove('on'));
  setSeg(null);
  lightArr[n-1].classList.add('on');
  setSeg(n);
  curText.textContent = n;
}
</script>
</body>
</html>
