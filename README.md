
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<title>RIDGELINE RALLY — Endless Stage</title>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/fontsource/fonts/chakra-petch@latest/latin-700-normal.css">
<link rel="stylesheet" href="https://cdn.jsdelivr.net/fontsource/fonts/chakra-petch@latest/latin-600-normal.css">
<link rel="stylesheet" href="https://cdn.jsdelivr.net/fontsource/fonts/chakra-petch@latest/latin-500-normal.css">
<link rel="stylesheet" href="https://cdn.jsdelivr.net/fontsource/fonts/barlow-condensed@latest/latin-700-normal.css">
<style>
:root{
  --accent:#ff7a18; --accent2:#ffd23f; --cyan:#63e6ff;
  --ink:#080a0d; --panel:rgba(11,14,19,.84); --line:rgba(255,255,255,.13);
  --good:#5ce07a; --bad:#ff4d4d;
  --sb:env(safe-area-inset-bottom,0px);
}
*{margin:0;padding:0;box-sizing:border-box;-webkit-tap-highlight-color:transparent}
html,body{width:100%;height:100%;overflow:hidden;background:#04060a;color:#fff;
  font-family:'Chakra Petch',system-ui,-apple-system,'Segoe UI',sans-serif;
  -webkit-user-select:none;user-select:none;touch-action:none;overscroll-behavior:none}
#app{position:fixed;inset:0}
canvas#gl{position:absolute;inset:0;width:100%;height:100%;display:block}
.hidden{display:none!important}
button{font-family:inherit}

/* ══ FX overlays ══ */
#vig{position:absolute;inset:0;pointer-events:none;opacity:0;mix-blend-mode:multiply;
  background:radial-gradient(ellipse at 50% 55%,rgba(0,0,0,0) 34%,rgba(0,0,0,.62) 100%);transition:opacity .16s linear}
#boostfx{position:absolute;inset:0;pointer-events:none;opacity:0;transition:opacity .12s linear;
  background:radial-gradient(ellipse at 50% 50%,rgba(255,140,30,0) 42%,rgba(255,110,20,.34) 100%)}
#damagefx{position:absolute;inset:0;pointer-events:none;opacity:0;transition:opacity .38s ease-out;
  background:radial-gradient(ellipse at 50% 50%,rgba(255,0,0,0) 38%,rgba(255,30,20,.6) 100%)}
#scan{position:absolute;inset:0;pointer-events:none;opacity:.045;
  background:repeating-linear-gradient(0deg,rgba(255,255,255,.7) 0 1px,transparent 1px 3px)}

/* ══ shared bits ══ */
.panel{background:var(--panel);border:1px solid var(--line);backdrop-filter:blur(10px);-webkit-backdrop-filter:blur(10px)}
.lbl{font-size:10px;letter-spacing:.22em;text-transform:uppercase;color:#8b95a1;font-weight:600}
.kicker{font-size:11px;letter-spacing:.42em;text-transform:uppercase;color:var(--accent);font-weight:700}
.btn{cursor:pointer;border:none;font-weight:700;font-size:16px;letter-spacing:.16em;text-transform:uppercase;
  padding:15px 40px;color:#0a0c10;background:linear-gradient(135deg,var(--accent2),var(--accent));
  clip-path:polygon(12px 0,100% 0,calc(100% - 12px) 100%,0 100%);
  transition:transform .12s,filter .12s;box-shadow:0 10px 34px rgba(255,122,24,.3)}
.btn:hover{transform:translateY(-2px) scale(1.02);filter:brightness(1.1)}
.btn:active{transform:translateY(1px) scale(.985)}
.btn.ghost{background:rgba(255,255,255,.05);color:#ccd4dd;border:1px solid var(--line);box-shadow:none;
  clip-path:polygon(10px 0,100% 0,calc(100% - 10px) 100%,0 100%);padding:13px 26px;font-size:12.5px}
.btn.ghost:hover{color:#fff;border-color:var(--accent);background:rgba(255,122,24,.12)}
.btn.ghost.on{border-color:var(--cyan);color:var(--cyan);background:rgba(99,230,255,.1)}
.btnrow{display:flex;gap:11px;justify-content:center;flex-wrap:wrap;align-items:center}
.iconBtn{width:38px;height:38px;display:grid;place-items:center;cursor:pointer;color:#c6ced8;
  background:rgba(12,16,22,.72);border:1px solid var(--line);font-size:16px;transition:.14s;
  clip-path:polygon(8px 0,100% 0,100% calc(100% - 8px),calc(100% - 8px) 100%,0 100%,0 8px)}
.iconBtn:hover{color:#fff;border-color:var(--accent);background:rgba(255,122,24,.16)}

/* ══ screens ══ */
.screen{position:absolute;inset:0;z-index:20;display:flex;align-items:center;justify-content:center;padding:18px;
  overflow:auto;background:radial-gradient(ellipse at 50% 42%,rgba(8,12,18,.6),rgba(3,5,9,.93))}
.card{width:min(760px,100%);text-align:center}
.brandline{display:flex;align-items:center;justify-content:center;gap:14px;margin-bottom:6px}
.brandline i{display:block;width:52px;height:2px;background:var(--accent)}
h1{font-family:'Barlow Condensed',sans-serif;font-size:clamp(48px,9.4vw,100px);line-height:.84;font-weight:700;
  text-transform:uppercase;margin:6px 0 4px;letter-spacing:-.01em}
h1 em{font-style:normal;color:var(--accent)}
.sub{color:#95a0ac;font-size:13.5px;letter-spacing:.06em;max-width:530px;margin:0 auto 22px;line-height:1.65}
.best{margin-top:20px;font-size:11px;letter-spacing:.24em;text-transform:uppercase;color:#78828e}
.best b{color:var(--accent2);font-family:'Barlow Condensed',sans-serif;font-size:20px;letter-spacing:.04em}

/* top utility bar */
#topbar{position:absolute;top:0;right:0;z-index:30;display:flex;gap:8px;padding:14px}
#topbar.hide{display:none}

/* ══ map select ══ */
#mapSelect{background:linear-gradient(100deg,rgba(3,5,9,.94) 0%,rgba(3,5,9,.86) 46%,rgba(3,5,9,.22) 78%,rgba(3,5,9,.5) 100%);
  align-items:stretch;justify-content:flex-start;padding:0}
.msLayout{display:grid;grid-template-columns:minmax(0,1fr) minmax(300px,372px);width:100%;height:100%;gap:0}
.msList{display:flex;flex-direction:column;min-height:0;border-right:1px solid var(--line);
  background:rgba(6,9,13,.72);backdrop-filter:blur(7px);-webkit-backdrop-filter:blur(7px)}
.msHead{padding:16px 18px 12px;border-bottom:1px solid var(--line);flex:0 0 auto}
.msHead .tt{display:flex;align-items:baseline;gap:12px;justify-content:space-between;flex-wrap:wrap}
.msHead h2{font-family:'Barlow Condensed',sans-serif;font-size:38px;font-weight:700;text-transform:uppercase;line-height:1}
.msHead .cnt{font-size:11px;letter-spacing:.2em;color:#7f8996;text-transform:uppercase}
.msTools{display:flex;gap:8px;margin-top:11px;flex-wrap:wrap;align-items:center}
#msSearch{flex:1;min-width:130px;background:rgba(255,255,255,.05);border:1px solid var(--line);color:#fff;
  padding:8px 11px;font-family:inherit;font-size:12.5px;letter-spacing:.06em;outline:none}
#msSearch:focus{border-color:var(--accent)}
.chip{cursor:pointer;padding:6px 12px;font-size:10.5px;letter-spacing:.14em;text-transform:uppercase;font-weight:700;
  color:#98a2ae;background:rgba(255,255,255,.04);border:1px solid var(--line);transition:.13s;white-space:nowrap}
.chip:hover{color:#fff;border-color:rgba(255,255,255,.3)}
.chip.on{color:#0a0c10;background:var(--accent2);border-color:var(--accent2)}
.msGrid{flex:1;min-height:0;overflow-y:auto;overflow-x:hidden;padding:14px;display:grid;gap:10px;
  grid-template-columns:repeat(auto-fill,minmax(158px,1fr));align-content:start;scrollbar-width:thin;
  scrollbar-color:rgba(255,255,255,.22) transparent}
.msGrid::-webkit-scrollbar{width:8px}
.msGrid::-webkit-scrollbar-thumb{background:rgba(255,255,255,.2)}
.mapCard{position:relative;cursor:pointer;text-align:left;padding:0;overflow:hidden;color:#fff;
  background:rgba(255,255,255,.045);border:1px solid var(--line);transition:transform .14s,border-color .14s,background .14s}
.mapCard:hover{transform:translateY(-3px);border-color:rgba(255,255,255,.32);background:rgba(255,255,255,.09)}
.mapCard.sel{border-color:var(--accent);background:rgba(255,122,24,.16);box-shadow:0 0 0 1px var(--accent),0 10px 28px rgba(255,122,24,.22)}
.mapCard canvas.thumb{display:block;width:100%;height:82px;background:linear-gradient(135deg,#151b23,#0c1015);
  image-rendering:auto}
.mcInfo{padding:8px 9px 9px}
.mcName{font-family:'Barlow Condensed',sans-serif;font-size:17px;font-weight:700;line-height:1.05;letter-spacing:.02em;
  white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.mcMeta{display:flex;justify-content:space-between;align-items:center;margin-top:4px;gap:6px}
.mcTheme{font-size:8.5px;letter-spacing:.16em;color:#8b95a1;text-transform:uppercase;white-space:nowrap;overflow:hidden}
.pips{font-size:9px;letter-spacing:1px;color:var(--accent);white-space:nowrap}
.pips i{font-style:normal;color:rgba(255,255,255,.18)}
.mcBadge{position:absolute;top:6px;left:6px;font-size:8px;letter-spacing:.14em;font-weight:700;padding:2px 6px;
  background:rgba(0,0,0,.66);border:1px solid var(--line);text-transform:uppercase}
.mcBadge.inf{color:var(--cyan);border-color:rgba(99,230,255,.5)}
.mcBadge.gh{color:var(--accent2);border-color:rgba(255,210,63,.5);left:auto;right:6px}
.mcBest{position:absolute;top:6px;right:6px;font-family:'Barlow Condensed',sans-serif;font-size:13px;font-weight:700;
  color:#fff;background:rgba(0,0,0,.6);padding:1px 6px;border-left:2px solid var(--accent)}

.msDetail{display:flex;flex-direction:column;justify-content:flex-end;padding:18px;gap:12px;min-height:0;overflow:auto}
.msCard{padding:16px;background:rgba(9,12,17,.66);clip-path:polygon(0 0,100% 0,100% calc(100% - 16px),calc(100% - 16px) 100%,0 100%)}
.msCard h3{font-family:'Barlow Condensed',sans-serif;font-size:34px;font-weight:700;text-transform:uppercase;line-height:.98}
.msTags{display:flex;gap:6px;flex-wrap:wrap;margin:8px 0 12px}
.tag{font-size:9px;letter-spacing:.14em;text-transform:uppercase;padding:3px 8px;border:1px solid var(--line);
  color:#a8b2bd;background:rgba(255,255,255,.04)}
.tag.hot{color:var(--accent);border-color:rgba(255,122,24,.5)}
.msStats{display:grid;grid-template-columns:1fr 1fr;gap:7px 14px;margin-bottom:12px}
.msStats div{display:flex;justify-content:space-between;font-size:11px;border-bottom:1px dashed rgba(255,255,255,.09);padding-bottom:3px}
.msStats span{color:#8b95a1;letter-spacing:.1em;text-transform:uppercase;font-size:9.5px}
.msStats b{font-family:'Barlow Condensed',sans-serif;font-size:15px;font-weight:700}
#msGhostLine{font-size:11px;color:var(--accent2);letter-spacing:.08em;margin-bottom:10px;min-height:15px}
.msBtns{display:flex;gap:8px;flex-wrap:wrap}
.msBtns .btn{flex:1;padding:13px 14px;font-size:13px;min-width:120px}
.msBtns .btn.ghost{flex:0 0 auto}
.msHint{font-size:10px;letter-spacing:.16em;color:#6f7986;text-transform:uppercase;text-align:center}

/* ══ settings ══ */
#settings{padding:0;align-items:stretch}
.setWrap{width:100%;height:100%;display:flex;align-items:stretch;justify-content:center;
  background:linear-gradient(180deg,rgba(3,5,9,.9),rgba(3,5,9,.96))}
.setCard{width:min(940px,100%);height:100%;display:flex;flex-direction:column;border-left:1px solid var(--line);
  border-right:1px solid var(--line);background:rgba(8,11,15,.9);backdrop-filter:blur(8px)}
.setHead{display:flex;align-items:center;gap:14px;padding:16px 20px;border-bottom:1px solid var(--line);flex:0 0 auto}
.setHead h2{font-family:'Barlow Condensed',sans-serif;font-size:34px;font-weight:700;text-transform:uppercase;flex:1;line-height:1}
.setBody{flex:1;overflow-y:auto;padding:18px 20px 30px;display:grid;gap:20px;
  grid-template-columns:repeat(auto-fit,minmax(280px,1fr));align-content:start;scrollbar-width:thin}
.setBody::-webkit-scrollbar{width:8px}.setBody::-webkit-scrollbar-thumb{background:rgba(255,255,255,.2)}
.setBody section{min-width:0}
.setBody h4{font-size:10.5px;letter-spacing:.26em;text-transform:uppercase;color:var(--accent);
  border-bottom:1px solid var(--line);padding-bottom:7px;margin-bottom:12px}
.seg{display:flex;gap:0;margin-bottom:13px;border:1px solid var(--line);flex-wrap:wrap}
.seg button{flex:1;min-width:56px;cursor:pointer;padding:9px 4px;background:transparent;border:none;color:#98a2ae;
  font-family:inherit;font-size:10.5px;font-weight:700;letter-spacing:.1em;text-transform:uppercase;
  border-right:1px solid var(--line);transition:.13s}
.seg button:last-child{border-right:none}
.seg button:hover{color:#fff;background:rgba(255,255,255,.06)}
.seg button.on{background:linear-gradient(135deg,var(--accent2),var(--accent));color:#0a0c10}
.row{display:flex;align-items:center;justify-content:space-between;gap:12px;padding:7px 0;font-size:12px;
  color:#c3ccd6;letter-spacing:.04em;border-bottom:1px dashed rgba(255,255,255,.07)}
.row b{font-family:'Barlow Condensed',sans-serif;color:var(--accent2);font-size:14px;font-weight:700;margin-left:5px}
.row input[type=range]{flex:0 0 132px;-webkit-appearance:none;appearance:none;height:4px;background:rgba(255,255,255,.16);outline:none;cursor:pointer}
.row input[type=range]::-webkit-slider-thumb{-webkit-appearance:none;width:15px;height:15px;background:var(--accent);
  border:2px solid #0a0c10;cursor:pointer;box-shadow:0 0 8px rgba(255,122,24,.6)}
.row input[type=range]::-moz-range-thumb{width:13px;height:13px;background:var(--accent);border:2px solid #0a0c10;cursor:pointer}
.sw{position:relative;flex:0 0 44px;height:22px;background:rgba(255,255,255,.13);border:1px solid var(--line);cursor:pointer;transition:.16s}
.sw::after{content:'';position:absolute;top:2px;left:2px;width:16px;height:16px;background:#8b95a1;transition:.16s}
.sw.on{background:rgba(255,122,24,.3);border-color:var(--accent)}
.sw.on::after{left:24px;background:var(--accent2)}
#sysInfo{font-size:11px;line-height:1.75;color:#8b95a1;letter-spacing:.04em;word-break:break-word}
#sysInfo b{color:#cdd5de;font-weight:600}
#fpsLive{color:var(--good);font-family:'Barlow Condensed',sans-serif;font-size:16px}

/* ══ HUD ══ */
#hud{position:absolute;inset:0;pointer-events:none;display:none;z-index:10}
#topLeft{position:absolute;top:14px;left:14px;padding:11px 15px 11px 13px;min-width:206px;
  border-left:3px solid var(--accent);
  clip-path:polygon(0 0,100% 0,100% calc(100% - 13px),calc(100% - 13px) 100%,0 100%)}
#scoreVal{font-family:'Barlow Condensed',sans-serif;font-size:38px;line-height:.92;font-weight:700}
#comboVal{display:inline-block;margin-left:7px;font-size:13px;font-weight:700;color:var(--accent2);
  vertical-align:6px;opacity:0;transition:opacity .2s}
#timeWrap{margin-top:8px}
#timeBarBg{width:100%;height:8px;background:rgba(255,255,255,.1);overflow:hidden;
  clip-path:polygon(0 0,100% 0,100% 100%,4px 100%)}
#timeBar{height:100%;width:100%;background:linear-gradient(90deg,var(--accent2),var(--accent));transition:width .1s linear}
#timeTxt{font-family:'Barlow Condensed',sans-serif;font-size:18px;font-weight:700;margin-top:1px;letter-spacing:.04em}
#kmhMini{display:none;font-family:'Barlow Condensed',sans-serif;font-size:26px;font-weight:700;color:#fff}
#kmhMini i{font-style:normal;font-size:10px;letter-spacing:.2em;color:#8b95a1;margin-left:3px}

#topRight{position:absolute;top:14px;right:14px;display:flex;flex-direction:column;align-items:flex-end;gap:9px}
#miniWrap{padding:7px;clip-path:polygon(13px 0,100% 0,100% 100%,0 100%,0 13px)}
#mini{display:block;width:128px;height:128px}
#miniLbl{position:absolute;bottom:9px;left:11px;font-size:8.5px;letter-spacing:.2em;color:#79838f;font-weight:600}
#statsBox{padding:8px 12px;display:flex;gap:15px;clip-path:polygon(0 0,100% 0,100% 100%,9px 100%,0 calc(100% - 9px))}
.stat{text-align:right}
.stat b{display:block;font-family:'Barlow Condensed',sans-serif;font-size:20px;font-weight:700;line-height:1}
.stat span{font-size:8.5px;letter-spacing:.16em;color:#8b95a1;text-transform:uppercase}
#ghostDelta{padding:5px 11px;font-family:'Barlow Condensed',sans-serif;font-size:19px;font-weight:700;
  border-left:3px solid var(--cyan);display:none}

#speedo{position:absolute;right:16px;bottom:16px;width:168px;height:168px}
#speedoCv{width:168px;height:168px;display:block}
#speedNum{position:absolute;left:0;right:0;bottom:32px;text-align:center}
#speedNum b{font-family:'Barlow Condensed',sans-serif;font-size:50px;font-weight:700;line-height:.85;display:block;
  text-shadow:0 2px 16px rgba(0,0,0,.75)}
#speedNum span{font-size:9.5px;letter-spacing:.28em;color:#98a2ae}
#gearNum{position:absolute;top:32px;left:0;right:0;text-align:center;font-family:'Barlow Condensed',sans-serif;
  font-size:21px;font-weight:700;color:var(--accent)}

#meters{position:absolute;left:16px;bottom:calc(18px + var(--sb));width:238px;display:flex;flex-direction:column;gap:8px}
.mlbl{font-size:9px;letter-spacing:.2em;color:#98a2ae;text-transform:uppercase;margin-bottom:3px;
  display:flex;justify-content:space-between;font-weight:600}
.mbg{height:12px;background:rgba(0,0,0,.55);border:1px solid var(--line);overflow:hidden;
  clip-path:polygon(0 0,100% 0,calc(100% - 7px) 100%,0 100%)}
.mfill{height:100%;width:100%;transition:width .12s linear}
#hpFill{background:linear-gradient(90deg,#ff3b30,#ff9f0a 55%,#5ce07a)}
#boostFill{background:linear-gradient(90deg,#1fa9ff,#63e6ff 60%,#dcfbff)}

#centerMsg{position:absolute;left:50%;top:25%;transform:translate(-50%,-50%);text-align:center;opacity:0}
#centerMsg b{font-family:'Barlow Condensed',sans-serif;font-size:clamp(50px,10vw,86px);font-weight:700;line-height:.9;
  display:block;text-shadow:0 6px 36px rgba(0,0,0,.85)}
#centerMsg span{font-size:14px;letter-spacing:.34em;text-transform:uppercase;color:var(--accent2)}
#toast{position:absolute;left:50%;top:14%;transform:translateX(-50%);display:flex;flex-direction:column;
  align-items:center;gap:5px}
.toastItem{font-family:'Barlow Condensed',sans-serif;font-size:25px;font-weight:700;letter-spacing:.05em;
  padding:2px 13px;background:rgba(0,0,0,.5);border-left:3px solid var(--accent2);white-space:nowrap;
  animation:tUp 1.5s ease-out forwards}
@keyframes tUp{0%{opacity:0;transform:translateY(13px) scale(.9)}14%{opacity:1;transform:translateY(0) scale(1)}
  74%{opacity:1}100%{opacity:0;transform:translateY(-28px) scale(1.02)}}
#airMeter{position:absolute;left:50%;bottom:27%;transform:translateX(-50%);text-align:center;opacity:0;transition:opacity .15s}
#airMeter b{font-family:'Barlow Condensed',sans-serif;font-size:32px;font-weight:700;color:var(--accent2);
  text-shadow:0 3px 18px rgba(0,0,0,.9);display:block;line-height:1}
#airMeter span{font-size:9.5px;letter-spacing:.3em;text-transform:uppercase}
#biomeTag{position:absolute;left:50%;top:14px;transform:translateX(-50%);padding:5px 16px;font-size:10px;
  letter-spacing:.3em;text-transform:uppercase;color:var(--cyan);border:1px solid rgba(99,230,255,.35);
  background:rgba(6,12,18,.6);display:none}

/* ══ touch controls ══ */
#touch{position:absolute;inset:0;display:none;pointer-events:none;z-index:14}
#touch.on{display:block}
.tb{position:absolute;pointer-events:auto;display:flex;align-items:center;justify-content:center;
  background:rgba(18,23,30,.42);border:1.5px solid rgba(255,255,255,.2);color:rgba(255,255,255,.85);
  font-weight:700;letter-spacing:.08em;border-radius:50%;opacity:.55;
  transition:opacity .1s,background .1s,transform .07s;backdrop-filter:blur(3px);-webkit-backdrop-filter:blur(3px)}
.tb.act{opacity:1;background:rgba(255,122,24,.6);transform:scale(.93);border-color:#fff}
#tL{left:calc(14px);bottom:calc(20px + var(--sb));width:66px;height:66px;font-size:24px}
#tR{left:calc(88px);bottom:calc(20px + var(--sb));width:66px;height:66px;font-size:24px}
#tGas{right:14px;bottom:calc(20px + var(--sb));width:82px;height:82px;font-size:14px}
#tBrk{right:calc(104px);bottom:calc(24px + var(--sb));width:60px;height:60px;font-size:11px}
#tNos{right:20px;bottom:calc(112px + var(--sb));width:54px;height:54px;font-size:10.5px;border-color:rgba(99,230,255,.55);color:var(--cyan)}
#tHnd{right:calc(84px);bottom:calc(96px + var(--sb));width:54px;height:54px;font-size:10px}

/* ══ results ══ */
#results{display:grid;grid-template-columns:repeat(auto-fit,minmax(112px,1fr));gap:9px;margin:22px 0 26px}
.res{padding:13px 7px;background:rgba(255,255,255,.045);border:1px solid var(--line)}
.res b{display:block;font-family:'Barlow Condensed',sans-serif;font-size:30px;font-weight:700;line-height:1}
.res.hi b{color:var(--accent)}
.res span{font-size:8.5px;letter-spacing:.2em;text-transform:uppercase;color:#8b95a1}
#overReason{font-family:'Barlow Condensed',sans-serif;font-size:clamp(36px,7vw,64px);font-weight:700;
  text-transform:uppercase;color:var(--bad);line-height:1;margin-bottom:2px}
#newBest{display:none;margin:0 auto 12px;width:fit-content;padding:5px 17px;background:var(--accent2);color:#0a0c10;
  font-weight:700;font-size:11.5px;letter-spacing:.24em;text-transform:uppercase}
#ctrlGrid{display:grid;grid-template-columns:repeat(auto-fit,minmax(142px,1fr));gap:8px;margin:22px 0 26px}
.ctrl{padding:11px 9px;background:rgba(255,255,255,.035);border:1px solid var(--line);
  clip-path:polygon(0 0,100% 0,100% calc(100% - 9px),calc(100% - 9px) 100%,0 100%)}
.ctrl kbd{display:inline-block;font-family:'Barlow Condensed',sans-serif;font-size:14px;font-weight:700;
  background:#1b2028;border:1px solid #2e3742;border-bottom-width:3px;padding:1px 8px;margin:0 1px;min-width:25px}
.ctrl p{font-size:9px;letter-spacing:.16em;text-transform:uppercase;color:#8b95a1;margin-top:6px}

/* ══ boot ══ */
#boot{position:absolute;inset:0;background:#04060a;z-index:80;display:flex;flex-direction:column;
  align-items:center;justify-content:center;gap:16px;transition:opacity .5s;padding:24px;text-align:center}
#boot.gone{opacity:0;pointer-events:none}
.spin{width:42px;height:42px;border:3px solid rgba(255,255,255,.12);border-top-color:var(--accent);
  border-radius:50%;animation:sp .9s linear infinite}
@keyframes sp{to{transform:rotate(360deg)}}
#bootTxt{font-size:11px;letter-spacing:.3em;text-transform:uppercase;color:#78828e}
#bootBar{width:min(260px,70vw);height:3px;background:rgba(255,255,255,.1);overflow:hidden}
#bootFill{height:100%;width:0%;background:linear-gradient(90deg,var(--accent2),var(--accent));transition:width .25s}
#errBox{max-width:520px;color:#ff8d8d;font-size:12.5px;line-height:1.65;display:none}

/* ══ responsive / mobile ══ */
@media (max-width:900px),(pointer:coarse){
  .msLayout{grid-template-columns:1fr;grid-template-rows:minmax(0,1fr) auto}
  .msList{border-right:none;border-bottom:1px solid var(--line)}
  .msDetail{padding:12px}
  .msCard{padding:13px}
  .msCard h3{font-size:26px}
  .msStats{grid-template-columns:1fr 1fr;gap:5px 10px;margin-bottom:9px}
  #mapSelect{background:linear-gradient(180deg,rgba(3,5,9,.5) 0%,rgba(3,5,9,.9) 42%,rgba(3,5,9,.96) 100%)}
  .msGrid{grid-template-columns:repeat(auto-fill,minmax(132px,1fr));padding:10px;gap:8px}
  .mapCard canvas.thumb{height:66px}
}
@media (max-width:760px),(pointer:coarse){
  #speedo{display:none}
  #kmhMini{display:block}
  #meters{width:170px;left:14px;bottom:auto;top:112px}
  #mini{width:88px;height:88px}
  #miniWrap{padding:5px}
  #topLeft{min-width:150px;padding:8px 12px 8px 10px}
  #scoreVal{font-size:27px}
  #timeTxt{font-size:15px}
  #statsBox{gap:10px;padding:6px 9px}
  .stat b{font-size:15px}
  #airMeter{bottom:auto;top:34%}
  #toast{top:22%}
  .toastItem{font-size:19px}
  #biomeTag{top:auto;bottom:calc(104px + var(--sb));font-size:9px;padding:4px 11px}
}
@media (max-height:520px){
  #meters{top:auto;bottom:calc(16px + var(--sb));width:150px}
  #miniWrap{display:none}
}
</style>
</head>
<body>
<div id="app">
  <canvas id="gl"></canvas>
  <div id="vig"></div><div id="boostfx"></div><div id="damagefx"></div><div id="scan"></div>

  <!-- ══════ HUD ══════ -->
  <div id="hud">
    <div id="topLeft" class="panel">
      <div class="lbl">Score</div>
      <div><span id="scoreVal">0</span><span id="comboVal">×1.0</span></div>
      <div id="kmhMini">0<i>KM/H</i></div>
      <div id="timeWrap">
        <div class="lbl" style="display:flex;justify-content:space-between">
          <span id="timeLbl">Stage Time</span><span id="cpLbl">CP 0</span></div>
        <div id="timeBarBg"><div id="timeBar"></div></div>
        <div id="timeTxt">40.0</div>
      </div>
    </div>

    <div id="biomeTag">TEMPERATE FOREST</div>

    <div id="topRight">
      <div id="miniWrap" class="panel"><canvas id="mini" width="256" height="256"></canvas><div id="miniLbl">NAV</div></div>
      <div id="statsBox" class="panel">
        <div class="stat"><b id="distVal">0</b><span>Meters</span></div>
        <div class="stat"><b id="coinVal">0</b><span>Coins</span></div>
        <div class="stat"><b id="airVal">0.0</b><span>Air</span></div>
      </div>
      <div id="ghostDelta" class="panel">GHOST +0m</div>
    </div>

    <div id="speedo">
      <canvas id="speedoCv" width="336" height="336"></canvas>
      <div id="gearNum">1</div>
      <div id="speedNum"><b id="kmh">0</b><span id="unitLbl">KM/H</span></div>
    </div>

    <div id="meters">
      <div><div class="mlbl"><span>Chassis</span><span id="hpTxt">100%</span></div>
        <div class="mbg"><div class="mfill" id="hpFill"></div></div></div>
      <div id="boostMeter"><div class="mlbl"><span>Nitrous</span><span id="boTxt">0%</span></div>
        <div class="mbg"><div class="mfill" id="boostFill" style="width:0%"></div></div></div>
    </div>

    <div id="centerMsg"><b id="cmBig">3</b><span id="cmSmall">Get Ready</span></div>
    <div id="toast"></div>
    <div id="airMeter"><b id="airTime">0.0s</b><span>Air Time</span></div>
  </div>

  <!-- ══════ TOUCH ══════ -->
  <div id="touch">
    <div class="tb" id="tL">◀</div><div class="tb" id="tR">▶</div>
    <div class="tb" id="tGas">GAS</div><div class="tb" id="tBrk">BRAKE</div>
    <div class="tb" id="tNos">NOS</div><div class="tb" id="tHnd">DRIFT</div>
  </div>

  <!-- ══════ TOP BAR ══════ -->
  <div id="topbar">
    <button class="iconBtn" id="btnSound" title="Mute (M)">🔊</button>
    <button class="iconBtn" id="btnFull" title="Fullscreen (F)">⛶</button>
    <button class="iconBtn" id="btnSet" title="Settings">⚙</button>
  </div>

  <!-- ══════ MAIN MENU ══════ -->
  <div class="screen" id="mainMenu">
    <div class="card">
      <div class="brandline"><i></i><span class="kicker">Endless Stage Protocol</span><i></i></div>
      <h1>RIDGE<em>LINE</em><br>RALLY</h1>
      <p class="sub">56 hand-tuned procedural stages, an endless morphing biome mode,
        and full ghost playback against your own best run. Pick your terrain. Beat your line.</p>
      <div class="btnrow">
        <button class="btn" id="btnPlay">Choose Stage</button>
        <button class="btn ghost" id="btnInfinite">Infinite Mode</button>
        <button class="btn ghost" id="btnSettings">Settings</button>
      </div>
      <div id="ctrlGrid">
        <div class="ctrl"><kbd>W</kbd><kbd>S</kbd><p>Throttle / Brake</p></div>
        <div class="ctrl"><kbd>A</kbd><kbd>D</kbd><p>Steer</p></div>
        <div class="ctrl"><kbd>SPACE</kbd><p>Handbrake Drift</p></div>
        <div class="ctrl"><kbd>SHIFT</kbd><p>Nitrous</p></div>
        <div class="ctrl"><kbd>C</kbd><kbd>F</kbd><p>Camera / Fullscreen</p></div>
        <div class="ctrl"><kbd>ESC</kbd><kbd>G</kbd><p>Pause / Ghost Cam</p></div>
      </div>
      <div class="best">Personal Best &nbsp;<b id="bestMenu">0</b></div>
    </div>
  </div>

  <!-- ══════ MAP SELECT ══════ -->
  <div class="screen hidden" id="mapSelect">
    <div class="msLayout">
      <aside class="msList">
        <div class="msHead">
          <div class="tt">
            <h2>Stage Select</h2>
            <span class="cnt" id="msCount">56 STAGES</span>
          </div>
          <div class="msTools">
            <input id="msSearch" type="text" placeholder="Search stage…" autocomplete="off">
            <button class="chip on" data-f="all">All</button>
            <button class="chip" data-f="easy">Easy</button>
            <button class="chip" data-f="hard">Hard</button>
            <button class="chip" data-f="ghost">Has Ghost</button>
          </div>
        </div>
        <div class="msGrid" id="msGrid"></div>
      </aside>
      <section class="msDetail">
        <div class="msHint">Live terrain preview · orbiting</div>
        <div class="msCard panel">
          <div class="lbl" id="msTheme">ARCTIC</div>
          <h3 id="msName">Glacier Run</h3>
          <div class="msTags" id="msTags"></div>
          <div class="msStats" id="msStats"></div>
          <div id="msGhostLine"></div>
          <div class="msBtns">
            <button class="btn" id="msStart">Launch Stage</button>
            <button class="btn ghost" id="msMode">Mode: Stage</button>
            <button class="btn ghost" id="msBack">Back</button>
          </div>
        </div>
      </section>
    </div>
  </div>

  <!-- ══════ SETTINGS ══════ -->
  <div class="screen hidden" id="settings">
    <div class="setWrap">
      <div class="setCard">
        <div class="setHead">
          <span class="kicker">System</span><h2>Settings</h2>
          <button class="btn ghost" id="setReset">Reset</button>
          <button class="iconBtn" id="setClose">✕</button>
        </div>
        <div class="setBody">
          <section>
            <h4>Graphics Preset</h4>
            <div class="seg" id="qualSeg">
              <button data-q="bare">Bare</button><button data-q="low">Low</button>
              <button data-q="med">Medium</button><button data-q="high">High</button><button data-q="best">Best</button>
            </div>
            <div class="row"><span>Auto-tune performance</span><div class="sw on" id="swAuto"></div></div>
            <div class="row"><span>Resolution scale<b id="vRes">100%</b></span><input type="range" id="setRes" min="50" max="100" step="5" value="100"></div>
            <div class="row"><span>Draw distance<b id="vDraw">100%</b></span><input type="range" id="setDraw" min="50" max="130" step="10" value="100"></div>
            <div class="row"><span>Shadows</span><div class="sw on" id="swShadow"></div></div>
            <div class="row"><span>Bloom</span><div class="sw on" id="swBloom"></div></div>
            <div class="row"><span>Skid marks</span><div class="sw on" id="swTrail"></div></div>
            <div class="row"><span>Particles<b id="vPart">100%</b></span><input type="range" id="setPart" min="0" max="150" step="25" value="100"></div>
          </section>
          <section>
            <h4>Gameplay &amp; Feel</h4>
            <div class="row"><span>Camera view</span>
              <div class="seg" style="flex:0 0 auto;margin:0" id="camSeg">
                <button data-c="0">Chase</button><button data-c="1">Hood</button><button data-c="2">Cine</button>
              </div></div>
            <div class="row"><span>Steering assist</span><div class="sw on" id="swAssist"></div></div>
            <div class="row"><span>Field of view<b id="vFov">64°</b></span><input type="range" id="setFov" min="52" max="88" step="2" value="64"></div>
            <div class="row"><span>Camera shake<b id="vShake">100%</b></span><input type="range" id="setShake" min="0" max="150" step="10" value="100"></div>
            <div class="row"><span>Speed units</span>
              <div class="seg" style="flex:0 0 auto;margin:0" id="unitSeg">
                <button data-u="kmh">KM/H</button><button data-u="mph">MPH</button>
              </div></div>
            <div class="row"><span>Record ghosts</span><div class="sw on" id="swRecord"></div></div>
          </section>
          <section>
            <h4>Audio</h4>
            <div class="row"><span>Master<b id="vVol">70%</b></span><input type="range" id="setVol" min="0" max="100" step="5" value="70"></div>
            <div class="row"><span>Engine<b id="vEng">80%</b></span><input type="range" id="setEng" min="0" max="100" step="5" value="80"></div>
            <div class="row"><span>Effects<b id="vSfx">80%</b></span><input type="range" id="setSfx" min="0" max="100" step="5" value="80"></div>
          </section>
          <section>
            <h4>Hardware</h4>
            <div id="sysInfo">Detecting…</div>
            <div class="row" style="margin-top:8px"><span>Live framerate</span><b id="fpsLive">--</b></div>
            <div class="btnrow" style="justify-content:flex-start;margin-top:12px">
              <button class="btn ghost" id="btnBench">Run 3s Benchmark</button>
            </div>
          </section>
        </div>
      </div>
    </div>
  </div>

  <!-- ══════ PAUSE ══════ -->
  <div class="screen hidden" id="pause">
    <div class="card">
      <div class="brandline"><i></i><span class="kicker">Stage Suspended</span><i></i></div>
      <h1>PAUSED</h1>
      <div class="btnrow" style="margin-top:20px">
        <button class="btn" id="btnResume">Resume</button>
        <button class="btn ghost" id="btnPauseSet">Settings</button>
        <button class="btn ghost" id="btnRestart">Restart</button>
        <button class="btn ghost" id="btnQuit">Abandon</button>
      </div>
    </div>
  </div>

  <!-- ══════ GAME OVER ══════ -->
  <div class="screen hidden" id="over">
    <div class="card">
      <div id="newBest">New Personal Best</div>
      <div id="overReason">Wrecked</div>
      <div class="kicker" style="margin-bottom:14px">Stage Report</div>
      <div id="results">
        <div class="res hi"><b id="rScore">0</b><span>Score</span></div>
        <div class="res"><b id="rDist">0</b><span>Meters</span></div>
        <div class="res"><b id="rCp">0</b><span>Checkpoints</span></div>
        <div class="res"><b id="rCoin">0</b><span>Coins</span></div>
        <div class="res"><b id="rTop">0</b><span>Top Speed</span></div>
        <div class="res"><b id="rAir">0</b><span>Best Air</span></div>
        <div class="res"><b id="rGhost">—</b><span>vs Ghost</span></div>
        <div class="res"><b id="rTime">0</b><span>Duration</span></div>
      </div>
      <div class="btnrow">
        <button class="btn" id="btnRetry">Run It Back</button>
        <button class="btn ghost" id="btnStageSel">Stage Select</button>
        <button class="btn ghost" id="btnMenu2">Main Menu</button>
      </div>
      <div class="best">Personal Best &nbsp;<b id="bestOver">0</b></div>
    </div>
  </div>

  <!-- ══════ BOOT ══════ -->
  <div id="boot">
    <div class="spin"></div>
    <div id="bootTxt">Building Terrain Engine…</div>
    <div id="bootBar"><div id="bootFill"></div></div>
    <div id="errBox"></div>
  </div>
</div>

<script type="importmap">
{"imports":{"three":"https://unpkg.com/three@0.160.0/build/three.module.js","three/addons/":"https://unpkg.com/three@0.160.0/examples/jsm/"}}
</script>

<script type="module">
import * as THREE from 'three';

/* ══════════════════════════════════════════════════════════════
   1 · UTILITIES
   ══════════════════════════════════════════════════════════════ */
const clamp=(v,a,b)=>v<a?a:(v>b?b:v);
const lerp=(a,b,t)=>a+(b-a)*t;
const damp=(a,b,l,dt)=>lerp(a,b,1-Math.exp(-l*dt));
const smoothstep=(e0,e1,x)=>{const t=clamp((x-e0)/(e1-e0),0,1);return t*t*(3-2*t);};
const TAU=Math.PI*2;
const $=id=>document.getElementById(id);
function rngFrom(seed){let s=(seed>>>0)||1;return function(){s=(Math.imul(s,1664525)+1013904223)>>>0;return s/4294967296;};}
function hashi(x,y){let h=Math.imul(x|0,374761393)+Math.imul(y|0,668265263);
  h=Math.imul(h^(h>>>13),1274126177);h^=h>>>16;return (h>>>0)/4294967296;}
function vnoise(x,y){
  const xi=Math.floor(x),yi=Math.floor(y),xf=x-xi,yf=y-yi;
  const u=xf*xf*(3-2*xf),v=yf*yf*(3-2*yf);
  const a=hashi(xi,yi),b=hashi(xi+1,yi),c=hashi(xi,yi+1),d=hashi(xi+1,yi+1);
  const ab=a+(b-a)*u, cd=c+(d-c)*u;
  return ab+(cd-ab)*v;
}
function fbm01(x,y,oct){let s=0,a=1,f=1,t=0;for(let i=0;i<oct;i++){s+=vnoise(x*f,y*f)*a;t+=a;a*=0.5;f*=2.03;}return s/t;}
function hex2rgb(h){return [((h>>16)&255)/255,((h>>8)&255)/255,(h&255)/255];}
function rgbHex(a){return (clamp(a[0],0,1)*255<<16)|(clamp(a[1],0,1)*255<<8)|(clamp(a[2],0,1)*255);}
function angDiff(a,b){let d=a-b;while(d>Math.PI)d-=TAU;while(d<-Math.PI)d+=TAU;return d;}

/* ══════════════════════════════════════════════════════════════
   2 · SETTINGS  (persisted)
   ══════════════════════════════════════════════════════════════ */
const PRESETS={
  bare:{label:'BARE',dpr:0.60,segs:44,span:340,fogM:0.55,cellR:4,shadow:false,shSize:512,bloom:false,
        part:0.25,trail:false,water:true,detail:0.55,msaa:0},
  low :{label:'LOW', dpr:0.78,segs:56,span:420,fogM:0.70,cellR:5,shadow:false,shSize:1024,bloom:false,
        part:0.50,trail:false,water:true,detail:0.75,msaa:0},
  med :{label:'MEDIUM',dpr:1.00,segs:72,span:500,fogM:0.85,cellR:6,shadow:true,shSize:1024,bloom:false,
        part:0.80,trail:true,water:true,detail:0.9,msaa:0},
  high:{label:'HIGH',dpr:1.30,segs:92,span:580,fogM:1.00,cellR:7,shadow:true,shSize:2048,bloom:true,
        part:1.00,trail:true,water:true,detail:1.0,msaa:2},
  best:{label:'BEST',dpr:2.00,segs:118,span:650,fogM:1.12,cellR:8,shadow:true,shSize:2048,bloom:true,
        part:1.50,trail:true,water:true,detail:1.15,msaa:4}
};
const DEF_SETTINGS={
  preset:'high', auto:true, resScale:100, draw:100, shadows:true, bloom:true, trail:true, part:100,
  cam:0, assist:true, fov:64, shake:100, units:'kmh', record:true,
  vol:70, eng:80, sfx:80, mode:'stage', mapIdx:0
};
let S=Object.assign({},DEF_SETTINGS);
try{const raw=localStorage.getItem('rr.settings');if(raw)S=Object.assign(S,JSON.parse(raw));}catch(e){}
function saveSettings(){try{localStorage.setItem('rr.settings',JSON.stringify(S));}catch(e){}}
function PQ(){return PRESETS[S.preset]||PRESETS.high;}

/* storage helpers */
function storeGet(k,d){try{const v=localStorage.getItem(k);return v===null?d:JSON.parse(v);}catch(e){return d;}}
function storeSet(k,v){try{localStorage.setItem(k,JSON.stringify(v));}catch(e){}}
let BESTS=storeGet('rr.best',{});

/* ══════════════════════════════════════════════════════════════
   3 · THEMES  (14 biomes)
   ══════════════════════════════════════════════════════════════ */
const THEMES={
arctic:{label:'ARCTIC',a:[15,6.0,1.8,.42],f:[.0030,.0098,.0295,.084],fr:.0016,ridge:.28,rp:2.2,ra:24,gain:1,
  water:-15,snowLine:.16,altLo:-22,altHi:40,treeD:.16,rockD:.32,bushD:.04,treeAlt:.55,tScale:.85,
  pal:{grass:0xd9e6ee,dry:0xc2d3de,rock:0x8d99a4,dark:0x636f7a,snow:0xffffff,sand:0xd2e1ea},
  tree:0xbdd2cd,trunk:0x5b5449,waterC:0x1b4f6d,fogC:0xd7e5ef,fogM:1.02,
  sky:[0x2a5f9e,0x9dc3e4,0xe6eff5],sunC:0xfff4e2,sunI:2.35,sunDir:[-.40,.60,.40],hemi:[0xcfe6ff,0x6d808b,.98],
  time:38,grip:.90,drag:1.02},
alpine:{label:'ALPINE',a:[21,7.6,2.1,.5],f:[.0032,.0106,.0310,.086],fr:.0017,ridge:.42,rp:2.0,ra:30,gain:1,
  water:-14,snowLine:.56,altLo:-20,altHi:52,treeD:.62,rockD:.58,bushD:.14,treeAlt:.66,tScale:1,
  pal:{grass:0x4c8c46,dry:0x8faa54,rock:0x7d756b,dark:0x544e47,snow:0xf4f8fa,sand:0xd9c48f},
  tree:0x2f6b3a,trunk:0x4e3a28,waterC:0x1f6f96,fogC:0xc9dae8,fogM:1,
  sky:[0x2f6fce,0x9ec6ea,0xe6d6b6],sunC:0xfff0d4,sunI:2.4,sunDir:[-.42,.62,.38],hemi:[0xbcd8ff,0x4a5a3a,.88],
  time:36,grip:1,drag:1},
tundra:{label:'TUNDRA',a:[11,4.6,1.4,.34],f:[.0026,.0090,.0280,.082],fr:.0014,ridge:.16,rp:2.4,ra:14,gain:1,
  water:-11,snowLine:.78,altLo:-16,altHi:28,treeD:.08,rockD:.34,bushD:.10,treeAlt:.5,tScale:.7,
  pal:{grass:0x7d8f62,dry:0x9aa877,rock:0x79726a,dark:0x565049,snow:0xeef3f5,sand:0xbfae86},
  tree:0x4a6b45,trunk:0x4c4034,waterC:0x2c6076,fogC:0xc3cfd4,fogM:1.08,
  sky:[0x4a7fae,0xa9c3d4,0xd8dcd2],sunC:0xf4f0e2,sunI:2.0,sunDir:[-.55,.44,.32],hemi:[0xb6cad6,0x5c6250,.95],
  time:40,grip:.94,drag:1},
forest:{label:'FOREST',a:[13,5.2,1.6,.4],f:[.0030,.0102,.0300,.085],fr:.0016,ridge:.20,rp:2.1,ra:16,gain:1,
  water:-12,snowLine:1.4,altLo:-18,altHi:34,treeD:1.0,rockD:.30,bushD:.30,treeAlt:.9,tScale:1,
  pal:{grass:0x3f7d3a,dry:0x6d9c40,rock:0x6f6a5f,dark:0x4b4a3f,snow:0xe8f0e2,sand:0xc6b183},
  tree:0x2c6b32,trunk:0x4a3524,waterC:0x24624f,fogC:0xb9cdb4,fogM:.92,
  sky:[0x3d7cb5,0xa6c8dc,0xdce0c4],sunC:0xfff2d0,sunI:2.2,sunDir:[-.34,.66,.44],hemi:[0xbdd8ee,0x3e5a30,.95],
  time:36,grip:1.02,drag:1},
autumn:{label:'AUTUMN',a:[14,5.4,1.7,.42],f:[.0029,.0100,.0305,.084],fr:.0015,ridge:.22,rp:2.1,ra:17,gain:1,
  water:-12,snowLine:1.4,altLo:-18,altHi:36,treeD:.92,rockD:.30,bushD:.22,treeAlt:.9,tScale:1,
  pal:{grass:0x7a8a3c,dry:0xa8913f,rock:0x74695c,dark:0x4f463c,snow:0xeae3d2,sand:0xcbb07e},
  tree:0xc86a20,trunk:0x4a3524,waterC:0x2b6478,fogC:0xd6c8ab,fogM:.95,
  sky:[0x4a79ad,0xc0c6cf,0xe8d2ac],sunC:0xffe6b8,sunI:2.15,sunDir:[-.52,.50,.42],hemi:[0xc8d6e8,0x6a5430,.9],
  time:36,grip:1,drag:1},
jungle:{label:'JUNGLE',a:[16,6.6,2.0,.48],f:[.0033,.0112,.0325,.088],fr:.0018,ridge:.30,rp:1.9,ra:22,gain:1,
  water:-9,snowLine:1.5,altLo:-14,altHi:38,treeD:1.35,rockD:.34,bushD:.55,treeAlt:.95,tScale:1.15,
  pal:{grass:0x2f6f30,dry:0x4f8a35,rock:0x5f6050,dark:0x3d4034,snow:0xdfeadb,sand:0xb5a375},
  tree:0x1f6b2c,trunk:0x3f2f1e,waterC:0x1d5b46,fogC:0xa8c4a4,fogM:.80,
  sky:[0x3a7fae,0x9fc0c4,0xd4dcc0],sunC:0xf6ffd8,sunI:2.05,sunDir:[-.30,.70,.48],hemi:[0xa8d8c0,0x2c4a22,1.0],
  time:38,grip:1.04,drag:1},
savanna:{label:'SAVANNA',a:[9,3.8,1.2,.3],f:[.0024,.0084,.0260,.078],fr:.0013,ridge:.14,rp:2.5,ra:12,gain:1,
  water:-13,snowLine:1.5,altLo:-16,altHi:26,treeD:.20,rockD:.22,bushD:.34,treeAlt:.85,tScale:1.05,
  pal:{grass:0xa89a44,dry:0xc9b45c,rock:0x8b7554,dark:0x63513a,snow:0xe6dcc2,sand:0xdcc48c},
  tree:0x6d8a3a,trunk:0x5c4429,waterC:0x2e7d8c,fogC:0xe0d0a8,fogM:1.05,
  sky:[0x4f88bb,0xc8d3cf,0xf0dcae],sunC:0xffe9b0,sunI:2.6,sunDir:[-.28,.72,.34],hemi:[0xd8e4f0,0x7a6a34,1.0],
  time:38,grip:.98,drag:1},
desert:{label:'DESERT',a:[12,5.0,1.5,.44],f:[.0028,.0108,.0360,.098],fr:.0014,ridge:.18,rp:2.6,ra:14,gain:1,
  water:-22,snowLine:1.5,altLo:-24,altHi:32,treeD:.02,rockD:.30,bushD:.05,treeAlt:.9,tScale:.9,
  pal:{grass:0xd8b978,dry:0xe8cd92,rock:0xb08a5c,dark:0x8a6a45,snow:0xf6ead0,sand:0xeed9a4},
  tree:0x7d8a4a,trunk:0x6a5334,waterC:0x2b7f9c,fogC:0xecd9ae,fogM:1.15,
  sky:[0x5f95c8,0xd6d3bd,0xf6e0b0],sunC:0xfff0c0,sunI:2.9,sunDir:[-.22,.78,.30],hemi:[0xe8e4d8,0x9c7c46,1.05],
  time:40,grip:.93,drag:1.04},
canyon:{label:'CANYON',a:[24,8.4,2.4,.55],f:[.0036,.0122,.0340,.090],fr:.0019,ridge:.55,rp:1.7,ra:36,gain:1,
  water:-24,snowLine:1.5,altLo:-26,altHi:56,treeD:.10,rockD:.75,bushD:.06,treeAlt:.8,tScale:.9,
  pal:{grass:0xb2663c,dry:0xc9834f,rock:0x8f4f33,dark:0x6a3a26,snow:0xe8c9a8,sand:0xd09a63},
  tree:0x5d7040,trunk:0x4a3524,waterC:0x2a6a78,fogC:0xdcb894,fogM:1.05,
  sky:[0x4d84bd,0xd0bda6,0xf2d5ab],sunC:0xffe3ae,sunI:2.7,sunDir:[-.46,.58,.30],hemi:[0xd8dcea,0x7a4c2c,.95],
  time:34,grip:.97,drag:1},
volcanic:{label:'VOLCANIC',a:[22,8.0,2.3,.52],f:[.0035,.0118,.0335,.092],fr:.0020,ridge:.60,rp:1.6,ra:38,gain:1,
  water:-8,snowLine:1.6,altLo:-14,altHi:58,treeD:.04,rockD:.85,bushD:.02,treeAlt:.7,tScale:.8,
  pal:{grass:0x4a4340,dry:0x5d5450,rock:0x3a3432,dark:0x241f1e,snow:0xc9c2bc,sand:0x6b5a4c},
  tree:0x3d4a34,trunk:0x2e2622,waterC:0xc8340c,fogC:0x6b5a58,fogM:.72,
  sky:[0x3a2c38,0x7a4a3e,0xd0703a],sunC:0xffb070,sunI:1.7,sunDir:[-.30,.42,.52],hemi:[0x8a6a6a,0x2a1c18,.8],
  time:34,grip:.95,drag:1},
coast:{label:'COAST',a:[8,3.4,1.1,.28],f:[.0026,.0092,.0285,.080],fr:.0013,ridge:.10,rp:2.4,ra:9,gain:1,
  water:-2,snowLine:1.5,altLo:-8,altHi:22,treeD:.28,rockD:.30,bushD:.18,treeAlt:.8,tScale:.95,
  pal:{grass:0x5f9a4c,dry:0x93b05a,rock:0x8b8578,dark:0x63605a,snow:0xeaf2f4,sand:0xf0dfb4},
  tree:0x35704a,trunk:0x55402c,waterC:0x1a86b4,fogC:0xcfe2ea,fogM:1.1,
  sky:[0x2f8fd0,0xa8d4ec,0xf0e6cc],sunC:0xfff6dc,sunI:2.7,sunDir:[-.36,.64,.46],hemi:[0xc4e4ff,0x6a7a50,1.0],
  time:38,grip:1,drag:1},
swamp:{label:'SWAMP',a:[6,2.6,.9,.24],f:[.0024,.0088,.0275,.079],fr:.0012,ridge:.08,rp:2.4,ra:7,gain:1,
  water:1.6,snowLine:1.5,altLo:-4,altHi:16,treeD:.95,rockD:.20,bushD:.55,treeAlt:.95,tScale:1.05,
  pal:{grass:0x4a6b3c,dry:0x6d7f42,rock:0x57544a,dark:0x3a3a30,snow:0xd8e0cc,sand:0x8d7f5a},
  tree:0x2f5a34,trunk:0x3c2f20,waterC:0x2f4a2c,fogC:0x9fae94,fogM:.66,
  sky:[0x4a6f7c,0x96a89a,0xc0c2a6],sunC:0xe8e8c0,sunI:1.75,sunDir:[-.44,.40,.48],hemi:[0x9fb8a8,0x354028,.95],
  time:42,grip:.92,drag:1.05},
badlands:{label:'BADLANDS',a:[17,6.4,1.9,.5],f:[.0034,.0126,.0380,.096],fr:.0018,ridge:.44,rp:1.8,ra:26,gain:1,
  water:-20,snowLine:1.5,altLo:-22,altHi:44,treeD:.06,rockD:.62,bushD:.10,treeAlt:.8,tScale:.85,
  pal:{grass:0x9c7a52,dry:0xb48d5e,rock:0x7a5f45,dark:0x594436,snow:0xdcc8ac,sand:0xc4a173},
  tree:0x6f7a44,trunk:0x513c26,waterC:0x3a7a86,fogC:0xd8c0a0,fogM:1.02,
  sky:[0x5b8ab8,0xcbbfa8,0xeed3a8],sunC:0xffe4b4,sunI:2.6,sunDir:[-.48,.56,.32],hemi:[0xd4d8e4,0x7a6040,.95],
  time:36,grip:.96,drag:1},
highland:{label:'HIGHLAND',a:[16,6.0,1.8,.44],f:[.0030,.0104,.0310,.086],fr:.0017,ridge:.32,rp:2.0,ra:22,gain:1,
  water:-13,snowLine:.86,altLo:-18,altHi:42,treeD:.22,rockD:.46,bushD:.26,treeAlt:.7,tScale:.9,
  pal:{grass:0x6d5f88,dry:0x8a6f9c,rock:0x6b6570,dark:0x4b4752,snow:0xe8e6f0,sand:0xa89278},
  tree:0x3d5f42,trunk:0x463628,waterC:0x24546e,fogC:0xbcb4c8,fogM:.92,
  sky:[0x4068a8,0x9fb0d0,0xd6c4d8],sunC:0xf0e4ff,sunI:2.1,sunDir:[-.50,.54,.42],hemi:[0xbcc4e8,0x4e4460,.95],
  time:36,grip:1,drag:1}
};

/* ══════════════════════════════════════════════════════════════
   4 · 56 STAGES
   ══════════════════════════════════════════════════════════════ */
const RAW=[
 ['Glacier Run','arctic',2,1.00,1.00,38],['Frostbite Flats','arctic',1,0.62,0.55,42],
 ['Polar Spine','arctic',4,1.30,1.55,34],['Whiteout Basin','arctic',3,0.95,1.10,36],
 ['Col du Renard','alpine',3,1.00,1.00,36],['Summit Ridge','alpine',5,1.42,1.70,32],
 ['Aiguille Noire','alpine',4,1.18,1.35,34],['Snowline Switchbacks','alpine',2,0.74,0.80,40],
 ['Permafrost Plain','tundra',1,0.80,0.60,44],['Lichen Steppe','tundra',2,1.00,0.90,42],
 ['Cold Hollow','tundra',3,1.24,1.25,38],['Caribou Flats','tundra',1,0.58,0.40,46],
 ['Pine Hollow','forest',1,0.70,0.70,44],['Green Cathedral','forest',2,1.00,1.00,40],
 ['Mossy Reach','forest',3,1.22,1.20,38],['Timberline Loop','forest',2,0.86,0.90,42],
 ['Ember Woods','autumn',2,0.88,0.90,42],['Harvest Ridge','autumn',3,1.14,1.20,38],
 ['Copper Canyon Road','autumn',4,1.34,1.45,36],['Fallen Leaf Pass','autumn',2,0.72,0.75,44],
 ['Verdant Deep','jungle',3,1.10,1.10,38],['Canopy Cut','jungle',2,0.80,0.80,42],
 ['Monsoon Trail','jungle',4,1.32,1.40,36],['Emerald Basin','jungle',3,0.96,1.05,40],
 ['Golden Plain','savanna',1,0.62,0.55,46],['Acacia Run','savanna',2,0.92,0.90,42],
 ['Dry Season','savanna',3,1.18,1.20,38],["Lion's Table",'savanna',2,0.78,0.85,44],
 ['Dune Sea','desert',2,0.95,0.90,42],['Salt Flat Sprint','desert',1,0.42,0.30,48],
 ['Mirage Basin','desert',3,1.20,1.25,38],['Erg Chebbi','desert',4,1.40,1.50,36],
 ['Red Cut','canyon',4,1.10,1.20,36],['Mesa Verde Stage','canyon',3,0.90,1.00,38],
 ['Slot Canyon','canyon',5,1.42,1.70,32],['Painted Wall','canyon',4,1.22,1.40,34],
 ['Ashfall','volcanic',4,1.12,1.25,36],['Caldera Rim','volcanic',5,1.45,1.80,32],
 ['Obsidian Flow','volcanic',3,0.92,1.05,38],['Cinder Stage','volcanic',4,1.24,1.45,34],
 ['Shoreline Sprint','coast',1,0.55,0.50,48],['Salt Air','coast',2,0.85,0.80,44],
 ['Cliffside Run','coast',4,1.30,1.40,36],['Tide Flats','coast',1,0.40,0.35,50],
 ['Bayou Black','swamp',2,0.75,0.70,44],['Mire Crossing','swamp',3,1.00,0.95,40],
 ['Cypress Slough','swamp',2,0.62,0.60,46],['Foggy Bottom','swamp',3,0.88,0.85,42],
 ['Broken Earth','badlands',3,1.05,1.10,38],['Gully Wash','badlands',4,1.30,1.45,34],
 ['Scarp Road','badlands',2,0.82,0.85,42],['Dust Bowl','badlands',3,0.95,1.20,40],
 ['Heather Hills','highland',2,0.84,0.85,42],['Purple Moor','highland',3,1.06,1.10,38],
 ['Wind Tor','highland',4,1.32,1.50,34],['Braemar Ridge','highland',3,1.00,1.25,40]
];

function makeParams(theme,opts){
  const T=THEMES[theme];
  const rnd=rngFrom((opts.seed|0)||1337);
  const s=opts.s===undefined?1:opts.s, r=opts.r===undefined?1:opts.r;
  const jf=()=>0.88+rnd()*0.26, jp=()=>rnd()*400;
  return {
    id:opts.id, name:opts.n||theme, theme, themeLabel:T.label, diff:opts.d||3,
    a0:T.a[0]*s,a1:T.a[1]*s,a2:T.a[2]*s,a3:T.a[3]*s,
    f0:T.f[0]*jf(),f1:T.f[1]*jf(),f2:T.f[2]*jf(),f3:T.f[3]*jf(),
    s0:jp(),s1:jp(),s2:jp(),s3:jp(),s4:jp(),s5:jp(),s6:jp(),s7:jp(),
    fr:T.fr*jf(),s8:jp(),s9:jp(),
    ridge:T.ridge*r,rp:T.rp,ra:T.ra*s*r,gain:T.gain,
    water:T.water,snowLine:T.snowLine,altLo:T.altLo,altHi:T.altHi,
    treeD:T.treeD*(opts.td||1),rockD:T.rockD*(opts.rd||1),bushD:T.bushD*(opts.bd||1),
    treeAlt:T.treeAlt,tScale:T.tScale,
    cG:hex2rgb(T.pal.grass),cD:hex2rgb(T.pal.dry),cR:hex2rgb(T.pal.rock),
    cK:hex2rgb(T.pal.dark),cS:hex2rgb(T.pal.snow),cA:hex2rgb(T.pal.sand),
    cT:hex2rgb(T.tree),cB:hex2rgb(T.trunk),cW:hex2rgb(T.waterC),cFog:hex2rgb(T.fogC),fogM:T.fogM,
    cSky0:hex2rgb(T.sky[0]),cSky1:hex2rgb(T.sky[1]),cSky2:hex2rgb(T.sky[2]),
    cSun:hex2rgb(T.sunC),sunI:T.sunI,sunDir:T.sunDir.slice(),
    hemiS:hex2rgb(T.hemi[0]),hemiG:hex2rgb(T.hemi[1]),hemiI:T.hemi[2],
    time:opts.t||T.time,cpTime:Math.max(7,(opts.t||T.time)*0.33),
    grip:T.grip,drag:T.drag,infinite:!!opts.infinite,label:T.label
  };
}
const NUMK=['a0','a1','a2','a3','f0','f1','f2','f3','s0','s1','s2','s3','s4','s5','s6','s7','fr','s8','s9',
 'ridge','rp','ra','gain','water','snowLine','altLo','altHi','treeD','rockD','bushD','treeAlt','tScale',
 'fogM','sunI','hemiI','time','cpTime','grip','drag'];
const V3K=['cG','cD','cR','cK','cS','cA','cT','cB','cW','cFog','cSky0','cSky1','cSky2','cSun','hemiS','hemiG','sunDir'];
function blendParams(A,B,w){
  const o={id:A.id,name:w<0.5?A.name:B.name,theme:w<0.5?A.theme:B.theme,
    themeLabel:w<0.5?A.label:B.label,label:w<0.5?A.label:B.label,diff:3,infinite:true};
  for(let i=0;i<NUMK.length;i++){const k=NUMK[i];o[k]=A[k]+(B[k]-A[k])*w;}
  for(let i=0;i<V3K.length;i++){const k=V3K[i];const a=A[k],b=B[k];
    o[k]=[a[0]+(b[0]-a[0])*w,a[1]+(b[1]-a[1])*w,a[2]+(b[2]-a[2])*w];}
  return o;
}

const MAPS=[];
for(let i=0;i<RAW.length;i++){
  const d=RAW[i];
  const p=makeParams(d[1],{id:'m'+i,n:d[0],d:d[2],s:d[3],r:d[4],t:d[5],seed:i*7919+13});
  p.idx=i;MAPS.push(p);
}
const INF_ORDER=['forest','desert','alpine','savanna','canyon','coast','autumn','badlands','volcanic','tundra','jungle','highland','swamp','arctic'];
const INF_PRESETS=[];
for(let i=0;i<INF_ORDER.length;i++)INF_PRESETS.push(makeParams(INF_ORDER[i],{id:'inf'+i,n:THEMES[INF_ORDER[i]].label,d:3,s:1,r:1,t:40,seed:i*3571+91,infinite:true}));
const INFINITE_MAP=makeParams('forest',{id:'infinite',n:'ENDLESS RIDGE',d:3,s:1,r:1,t:0,seed:4242,infinite:true});
INFINITE_MAP.idx=-1;INFINITE_MAP.infinite=true;

/* ── terrain sampling ── */
let MAP=MAPS[0];
let _pKey=NaN,_pP=null,_terrVer=0;
function paramsAt(z){
  if(!MAP.infinite)return MAP;
  const key=Math.floor(z/20);
  if(key===_pKey)return _pP;
  const zc=key*20,BAND=980,FADE=300;
  const fi=Math.floor(zc/BAND),local=zc-fi*BAND;
  const w=smoothstep(BAND-FADE,BAND,local);
  const n=INF_PRESETS.length;
  const A=INF_PRESETS[((fi%n)+n)%n],B=INF_PRESETS[(((fi+1)%n)+n)%n];
  _pKey=key;_pP=w<0.001?A:(w>0.999?B:blendParams(A,B,w));
  return _pP;
}
function invalidateParams(){_pKey=NaN;_pP=null;_terrVer++;}
function groundHeight(x,z){
  const p=paramsAt(z);
  let h=0;
  h+=(vnoise(x*p.f0+p.s0,z*p.f0+p.s1)-0.5)*2*p.a0;
  h+=(vnoise(x*p.f1+p.s2,z*p.f1+p.s3)-0.5)*2*p.a1;
  h+=(vnoise(x*p.f2+p.s4,z*p.f2+p.s5)-0.5)*2*p.a2;
  h+=(vnoise(x*p.f3+p.s6,z*p.f3+p.s7)-0.5)*2*p.a3;
  if(p.ridge>0.001){
    const r=vnoise(x*p.fr+p.s8,z*p.fr+p.s9);
    const t=clamp(r*(1+p.ridge)-p.ridge*0.60,0,1);
    h+=Math.pow(t,p.rp)*p.ra;
  }
  return h*p.gain;
}
const _nv=new THREE.Vector3();
function groundNormal(x,z,out){
  const e=1.15;
  const hl=groundHeight(x-e,z),hr=groundHeight(x+e,z),hd=groundHeight(x,z-e),hu=groundHeight(x,z+e);
  return out.set(hl-hr,2*e,hd-hu).normalize();
}
function groundSlope(x,z){
  const e=1.6;
  const gx=(groundHeight(x+e,z)-groundHeight(x-e,z))/(2*e);
  const gz=(groundHeight(x,z+e)-groundHeight(x,z-e))/(2*e);
  return Math.sqrt(gx*gx+gz*gz);
}

/* ══════════════════════════════════════════════════════════════
   5 · RENDERER
   ══════════════════════════════════════════════════════════════ */
const canvas=$('gl');
let renderer=null;
try{
  renderer=new THREE.WebGLRenderer({canvas,antialias:PQ().msaa>0,powerPreference:'high-performance',stencil:false});
}catch(e){
  $('errBox').style.display='block';
  $('errBox').textContent='WebGL could not be initialised on this device/browser.';
  document.querySelector('.spin').style.display='none';
  throw e;
}
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(1);
renderer.outputColorSpace=THREE.SRGBColorSpace;
renderer.toneMapping=THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure=1.06;
renderer.shadowMap.enabled=true;
renderer.shadowMap.type=THREE.PCFSoftShadowMap;

const scene=new THREE.Scene();
scene.fog=new THREE.Fog(0xc9d8e6,90,300);
const camera=new THREE.PerspectiveCamera(S.fov,innerWidth/innerHeight,0.35,4200);
camera.position.set(0,12,-18);

const hemi=new THREE.HemisphereLight(0xbcd8ff,0x4a5a3a,0.9);scene.add(hemi);
const sun=new THREE.DirectionalLight(0xfff0d4,2.4);
sun.castShadow=true;
sun.shadow.camera.left=-72;sun.shadow.camera.right=72;
sun.shadow.camera.top=72;sun.shadow.camera.bottom=-72;
sun.shadow.camera.near=1;sun.shadow.camera.far=340;
sun.shadow.bias=-0.0009;sun.shadow.normalBias=0.55;
scene.add(sun);scene.add(sun.target);
const fillL=new THREE.DirectionalLight(0x88aaff,0.4);fillL.position.set(.6,.35,-.7);scene.add(fillL);
const sunDirV=new THREE.Vector3(-.42,.62,.38).normalize();

/* sky dome */
const skyU={uTop:{value:new THREE.Color()},uMid:{value:new THREE.Color()},uBot:{value:new THREE.Color()},
  uSun:{value:new THREE.Color()},uSunDir:{value:sunDirV.clone()},uHaze:{value:new THREE.Color()}};
const sky=new THREE.Mesh(new THREE.SphereGeometry(1,32,20),new THREE.ShaderMaterial({
  uniforms:skyU,side:THREE.BackSide,depthWrite:false,fog:false,
  vertexShader:'varying vec3 vD;void main(){vD=normalize(position);gl_Position=projectionMatrix*modelViewMatrix*vec4(position,1.0);}',
  fragmentShader:'uniform vec3 uTop,uMid,uBot,uSun,uSunDir,uHaze;varying vec3 vD;'+
  'void main(){vec3 d=normalize(vD);float h=d.y;'+
  'vec3 c=mix(uMid,uTop,pow(clamp(h,0.0,1.0),0.55));'+
  'c=mix(uBot,c,smoothstep(-0.18,0.06,h));'+
  'float sd=max(dot(d,normalize(uSunDir)),0.0);'+
  'c+=uSun*pow(sd,460.0)*3.4;c+=uSun*pow(sd,11.0)*0.30;c+=uSun*pow(sd,2.6)*0.07;'+
  'c=mix(c,uHaze,smoothstep(0.22,-0.02,h)*0.55);'+
  'gl_FragColor=vec4(c,1.0);}'
}));
sky.scale.setScalar(2800);sky.frustumCulled=false;sky.renderOrder=-1000;scene.add(sky);

/* ══════════════════════════════════════════════════════════════
   6 · TERRAIN MESH (snapping infinite grid)
   ══════════════════════════════════════════════════════════════ */
let TSEG=90,TCELL=5,TSIZE=450;
let terrainGeo=null,terrainMesh=null,TLX=null,TLZ=null,gridH=null,colorArr=null;
const terrainMat=new THREE.MeshStandardMaterial({vertexColors:true,flatShading:true,roughness:.96,metalness:0});
const _tc=new THREE.Color();
function buildTerrain(){
  const q=PQ();
  TSEG=Math.max(24,Math.round(q.segs*(S.draw/100)));
  TSIZE=q.span*(S.draw/100);
  TCELL=TSIZE/TSEG;
  if(terrainMesh){scene.remove(terrainMesh);}
  if(terrainGeo)terrainGeo.dispose();
  const g=new THREE.PlaneGeometry(TSIZE,TSIZE,TSEG,TSEG);
  g.rotateX(-Math.PI/2);
  const n=(TSEG+1)*(TSEG+1),pa=g.attributes.position.array;
  TLX=new Float32Array(n);TLZ=new Float32Array(n);
  for(let i=0;i<n;i++){TLX[i]=pa[i*3];TLZ[i]=pa[i*3+2];}
  g.setAttribute('color',new THREE.BufferAttribute(new Float32Array(n*3),3));
  gridH=new Float32Array(n);colorArr=g.attributes.color.array;
  g.attributes.position.setUsage(THREE.DynamicDrawUsage);
  g.attributes.color.setUsage(THREE.DynamicDrawUsage);
  terrainGeo=g;
  terrainMesh=new THREE.Mesh(g,terrainMat);
  terrainMesh.receiveShadow=true;terrainMesh.frustumCulled=false;
  scene.add(terrainMesh);
  lastSnapX=1e9;lastSnapZ=1e9;
}
let lastSnapX=1e9,lastSnapZ=1e9;
function updateTerrain(cx,cz,force){
  const sx=Math.round(cx/TCELL)*TCELL,sz=Math.round(cz/TCELL)*TCELL;
  if(!force&&sx===lastSnapX&&sz===lastSnapZ)return;
  lastSnapX=sx;lastSnapZ=sz;
  terrainMesh.position.x=sx;terrainMesh.position.z=sz;
  const pa=terrainGeo.attributes.position.array,n=gridH.length,W=TSEG+1;
  for(let i=0;i<n;i++){
    const h=groundHeight(sx+TLX[i],sz+TLZ[i]);
    gridH[i]=h;pa[i*3+1]=h;
  }
  const p=paramsAt(cz);
  const cG=_tc.setRGB(p.cG[0],p.cG[1],p.cG[2],THREE.SRGBColorSpace).clone();
  const cD=new THREE.Color().setRGB(p.cD[0],p.cD[1],p.cD[2],THREE.SRGBColorSpace);
  const cR=new THREE.Color().setRGB(p.cR[0],p.cR[1],p.cR[2],THREE.SRGBColorSpace);
  const cK=new THREE.Color().setRGB(p.cK[0],p.cK[1],p.cK[2],THREE.SRGBColorSpace);
  const cS=new THREE.Color().setRGB(p.cS[0],p.cS[1],p.cS[2],THREE.SRGBColorSpace);
  const cA=new THREE.Color().setRGB(p.cA[0],p.cA[1],p.cA[2],THREE.SRGBColorSpace);
  const altLo=p.altLo,altSpan=Math.max(1,p.altHi-p.altLo),wLevel=p.water;
  const tA=new THREE.Color(),tB=new THREE.Color();
  for(let iy=0;iy<W;iy++){
    const wz=sz+TLZ[iy*W];
    const pp=MAP.infinite?paramsAt(wz):p;
    const pcG=tA.setRGB(pp.cG[0],pp.cG[1],pp.cG[2],THREE.SRGBColorSpace);
    for(let ix=0;ix<W;ix++){
      const i=iy*W+ix,h=gridH[i];
      const hx=gridH[iy*W+(ix<W-1?ix+1:ix)]-gridH[iy*W+(ix>0?ix-1:ix)];
      const hz=gridH[(iy<W-1?iy+1:iy)*W+ix]-gridH[(iy>0?iy-1:iy)*W+ix];
      const slope=Math.sqrt(hx*hx+hz*hz)/(2*TCELL);
      const wx=sx+TLX[i];
      const biome=vnoise(wx*0.0042+310,wz*0.0042-190);
      const patch=vnoise(wx*0.026-70,wz*0.026+40);
      const hN=clamp((h-altLo)/altSpan,0,1);
      const gA=(pp.cG===pcG)?pcG:cG;
      _tc.copy(gA).lerp(cD,clamp(biome*1.55-0.74,0,1)*0.85);
      _tc.lerp(new THREE.Color(0,0,0),clamp(patch*0.9-0.42,0,1)*0.22);
      tB.copy(cR).lerp(cK,clamp(patch*1.25-0.42,0,1));
      _tc.lerp(tB,smoothstep(0.40,0.76,hN));
      const sn=smoothstep(pp.snowLine,Math.min(0.99,pp.snowLine+0.20),hN)*clamp(1.15-slope*1.5,0,1);
      if(sn>0.001)_tc.lerp(cS,sn);
      _tc.lerp(cK,smoothstep(0.44,0.88,slope)*0.82);
      _tc.lerp(cA,smoothstep(wLevel+3.4,wLevel-0.8,h)*0.92);
      colorArr[i*3]=_tc.r;colorArr[i*3+1]=_tc.g;colorArr[i*3+2]=_tc.b;
    }
  }
  terrainGeo.attributes.position.needsUpdate=true;
  terrainGeo.attributes.color.needsUpdate=true;
  terrainGeo.computeVertexNormals();
}

/* water */
const waterMat=new THREE.MeshStandardMaterial({color:0x1d6f9c,roughness:.13,metalness:.4,transparent:true,opacity:.86});
const water=new THREE.Mesh(new THREE.PlaneGeometry(3000,3000,1,1),waterMat);
water.rotation.x=-Math.PI/2;water.frustumCulled=false;scene.add(water);

/* ══════════════════════════════════════════════════════════════
   7 · PROPS  (deterministic streamed cells, instanced)
   ══════════════════════════════════════════════════════════════ */
const CELL=44;
let cells=new Map(),worldObjects=[],propsDirty=true;
const ck=(a,b)=>a+','+b;

const trunkGeo=new THREE.CylinderGeometry(.26,.46,2.6,5,1);trunkGeo.translate(0,1.3,0);
const canopyGeo=(function(){
  const A=new THREE.ConeGeometry(2.25,4.6,7);A.translate(0,4.4,0);
  const B=new THREE.ConeGeometry(1.55,3.4,7);B.translate(0,6.7,0);
  const a=A.toNonIndexed(),b=B.toNonIndexed();
  const pa=a.attributes.position.array,pb=b.attributes.position.array;
  const na=a.attributes.normal.array,nb=b.attributes.normal.array;
  const pos=new Float32Array(pa.length+pb.length);pos.set(pa,0);pos.set(pb,pa.length);
  const nor=new Float32Array(na.length+nb.length);nor.set(na,0);nor.set(nb,na.length);
  const g=new THREE.BufferGeometry();
  g.setAttribute('position',new THREE.BufferAttribute(pos,3));
  g.setAttribute('normal',new THREE.BufferAttribute(nor,3));
  A.dispose();B.dispose();a.dispose();b.dispose();return g;
})();
function jitterGeo(g,amt){
  const p=g.attributes.position,r=rngFrom(9182734),seen=new Map();
  for(let i=0;i<p.count;i++){
    const k=p.getX(i).toFixed(3)+'|'+p.getY(i).toFixed(3)+'|'+p.getZ(i).toFixed(3);
    if(!seen.has(k))seen.set(k,[(r()-.5)*amt,(r()-.5)*amt,(r()-.5)*amt]);
    const j=seen.get(k);p.setXYZ(i,p.getX(i)+j[0],p.getY(i)+j[1],p.getZ(i)+j[2]);
  }
  g.computeVertexNormals();
}
const rockGeo=new THREE.IcosahedronGeometry(1.55,0);jitterGeo(rockGeo,.30);rockGeo.translate(0,.6,0);
const bushGeo=new THREE.IcosahedronGeometry(1.0,0);jitterGeo(bushGeo,.24);bushGeo.translate(0,.55,0);

const MAX_TREE=3600,MAX_ROCK=2400,MAX_BUSH=2000;
const trunkMat=new THREE.MeshStandardMaterial({color:0xffffff,roughness:.95,flatShading:true});
const canopyMat=new THREE.MeshStandardMaterial({color:0xffffff,roughness:.9,flatShading:true});
const rockMat=new THREE.MeshStandardMaterial({color:0xffffff,roughness:.95,metalness:.03,flatShading:true});
const bushMat=new THREE.MeshStandardMaterial({color:0xffffff,roughness:.92,flatShading:true});
const trunkIM=new THREE.InstancedMesh(trunkGeo,trunkMat,MAX_TREE);
const canopyIM=new THREE.InstancedMesh(canopyGeo,canopyMat,MAX_TREE);
const rockIM=new THREE.InstancedMesh(rockGeo,rockMat,MAX_ROCK);
const bushIM=new THREE.InstancedMesh(bushGeo,bushMat,MAX_BUSH);
[trunkIM,canopyIM,rockIM,bushIM].forEach(m=>{m.castShadow=true;m.receiveShadow=true;m.frustumCulled=false;m.count=0;
  m.instanceMatrix.setUsage(THREE.DynamicDrawUsage);scene.add(m);});
const _m4=new THREE.Matrix4(),_q=new THREE.Quaternion(),_v3=new THREE.Vector3(),_sv=new THREE.Vector3();
const _eul=new THREE.Euler(),_col=new THREE.Color();

function generateCell(cx,cz){
  const seed=(Math.imul(cx,73856093)^Math.imul(cz,19349663)^0x9e3779b9)>>>0;
  const rnd=rngFrom(seed),objs=[];
  const x0=cx*CELL,z0=cz*CELL,ccx=x0+CELL*.5,ccz=z0+CELL*.5;
  if(!MAP.infinite&&Math.abs(ccx)<70&&Math.abs(ccz)<70)return objs;
  const p=paramsAt(ccz);
  const h=groundHeight(ccx,ccz);
  if(h<p.water+0.8)return objs;
  const forestN=fbm01(ccx*.0058+300,ccz*.0058-200,3);
  const rockyN=fbm01(ccx*.0089-140,ccz*.0089+90,2);
  const diffMul=1+(MAP.infinite?Math.min(1.1,Math.abs(ccz)/9000):(p.diff-3)*0.08);
  const treeP=(forestN-0.46)*2.3*p.treeD*diffMul;
  const rockP=(rockyN-0.45)*1.7*p.rockD*diffMul;
  const tries=Math.floor(CELL*CELL/30);
  let placed=0;
  for(let t=0;t<tries;t++){
    const px=x0+rnd()*CELL,pz=z0+rnd()*CELL;
    const ph=groundHeight(px,pz);
    if(ph<p.water+1.2)continue;
    if(groundSlope(px,pz)>0.78)continue;
    const alt=clamp((ph-p.altLo)/Math.max(1,p.altHi-p.altLo),0,1);
    const rr=rnd();let o=null;
    if(rr<treeP&&alt<p.treeAlt){
      const sc=(0.70+rnd()*0.88)*p.tScale;
      o={t:'tree',x:px,z:pz,y:ph,s:sc,rot:rnd()*TAU,col:rnd(),r:0.95*sc};
    }else if(rr<treeP+rockP){
      const sc=0.5+rnd()*1.55;
      o={t:'rock',x:px,z:pz,y:ph,s:sc,rot:rnd()*TAU,col:rnd(),r:1.32*sc};
    }else if(rr<treeP+rockP+p.bushD*0.22&&alt<0.72){
      const sc=0.6+rnd()*0.95;
      o={t:'bush',x:px,z:pz,y:ph,s:sc,rot:rnd()*TAU,col:rnd(),r:0.7*sc};
    }
    if(o){objs.push(o);if(++placed>17)break;}
  }
  if(rnd()<0.44){
    const bx=x0+8+rnd()*(CELL-16),bz=z0+8+rnd()*(CELL-16);
    const ang=rnd()*TAU,arc=(rnd()-.5)*1.5,nn=5+Math.floor(rnd()*4);
    for(let i=0;i<nn;i++){
      const a=ang+(i/(nn-1)-.5)*arc;
      const px=bx+Math.cos(a)*i*4.6,pz=bz+Math.sin(a)*i*4.6;
      const ph=groundHeight(px,pz);
      if(ph<p.water+1)continue;
      objs.push({t:'coin',x:px,z:pz,y:ph+1.5,s:1,rot:rnd()*TAU,col:0,r:2.0,got:false});
    }
  }
  if(rnd()<0.17){
    const px=x0+10+rnd()*(CELL-20),pz=z0+10+rnd()*(CELL-20);
    const ph=groundHeight(px,pz);
    if(ph>p.water+1.6&&groundSlope(px,pz)<0.42)
      objs.push({t:'pad',x:px,z:pz,y:ph+.12,s:1,rot:rnd()*TAU,col:0,r:4.4,cool:0,mesh:null});
  }
  return objs;
}
function rebuildFlat(){
  worldObjects=[];
  for(const arr of cells.values())for(let i=0;i<arr.length;i++)worldObjects.push(arr[i]);
  propsDirty=true;
}
function refreshCells(cx,cz){
  const bcx=Math.floor(cx/CELL),bcz=Math.floor(cz/CELL),R=PQ().cellR;
  let changed=false;
  const R2=R*R+2;
  for(let dz=-R;dz<=R;dz++){
    for(let dx=-R;dx<=R;dx++){
      if(dx*dx+dz*dz>R2)continue;
      const k=ck(bcx+dx,bcz+dz);
      if(!cells.has(k)){cells.set(k,generateCell(bcx+dx,bcz+dz));changed=true;}
    }
  }
  const lim=R+2;
  for(const k of Array.from(cells.keys())){
    const i=k.indexOf(','),px=+k.slice(0,i),pz=+k.slice(i+1);
    if(Math.abs(px-bcx)>lim||Math.abs(pz-bcz)>lim){cells.delete(k);changed=true;}
  }
  if(changed)rebuildFlat();
}
function clearArea(x,z,rad){
  let any=false,r2=rad*rad;
  for(const arr of cells.values())for(let i=arr.length-1;i>=0;i--){
    const o=arr[i];
    const dx=o.x-x,dz=o.z-z;
    if(dx*dx+dz*dz<r2&&(o.t==='tree'||o.t==='rock'||o.t==='bush')){arr.splice(i,1);any=true;}
  }
  if(any)rebuildFlat();
}
function rebuildProps(){
  if(!propsDirty)return;propsDirty=false;
  const p=paramsAt(car.pos.z);
  let ti=0,ri=0,bi=0;
  const tg=p.cT,tb=p.cB,sc=PQ().detail;
  for(let i=0;i<worldObjects.length;i++){
    const o=worldObjects[i];
    if(o.t==='tree'&&ti<MAX_TREE){
      _v3.set(o.x,o.y-0.15,o.z);_eul.set(0,o.rot,0);_q.setFromEuler(_eul);
      _sv.set(o.s,o.s*(0.85+o.col*0.35),o.s);_m4.compose(_v3,_q,_sv);
      trunkIM.setMatrixAt(ti,_m4);canopyIM.setMatrixAt(ti,_m4);
      const g=(0.74+o.col*0.46)*sc;
      _col.setRGB(tb[0]*g,tb[1]*g,tb[2]*g,THREE.SRGBColorSpace);trunkIM.setColorAt(ti,_col);
      _col.setRGB(tg[0]*g,tg[1]*g,tg[2]*g,THREE.SRGBColorSpace);canopyIM.setColorAt(ti,_col);
      ti++;
    }else if(o.t==='rock'&&ri<MAX_ROCK){
      _v3.set(o.x,o.y-0.25,o.z);_eul.set(o.col*.5,o.rot,o.col*.3);_q.setFromEuler(_eul);
      _sv.set(o.s*(.8+o.col*.5),o.s*(.65+o.col*.4),o.s*(.85+o.col*.4));
      _m4.compose(_v3,_q,_sv);rockIM.setMatrixAt(ri,_m4);
      const v=(.6+o.col*.44)*sc;
      _col.setRGB(p.cR[0]*v,p.cR[1]*v,p.cR[2]*v,THREE.SRGBColorSpace);rockIM.setColorAt(ri,_col);ri++;
    }else if(o.t==='bush'&&bi<MAX_BUSH){
      _v3.set(o.x,o.y-0.15,o.z);_eul.set(0,o.rot,0);_q.setFromEuler(_eul);
      _sv.set(o.s*(1.1+o.col*.4),o.s*(.8+o.col*.5),o.s*(1.1+o.col*.4));
      _m4.compose(_v3,_q,_sv);bushIM.setMatrixAt(bi,_m4);
      const g=(.7+o.col*.5)*sc;
      _col.setRGB(tg[0]*g*.85,tg[1]*g*1.05,tg[2]*g*.8,THREE.SRGBColorSpace);bushIM.setColorAt(bi,_col);bi++;
    }
  }
  trunkIM.count=ti;canopyIM.count=ti;rockIM.count=ri;bushIM.count=bi;
  trunkIM.instanceMatrix.needsUpdate=true;canopyIM.instanceMatrix.needsUpdate=true;
  rockIM.instanceMatrix.needsUpdate=true;bushIM.instanceMatrix.needsUpdate=true;
  if(trunkIM.instanceColor)trunkIM.instanceColor.needsUpdate=true;
  if(canopyIM.instanceColor)canopyIM.instanceColor.needsUpdate=true;
  if(rockIM.instanceColor)rockIM.instanceColor.needsUpdate=true;
  if(bushIM.instanceColor)bushIM.instanceColor.needsUpdate=true;
}

/* ══════════════════════════════════════════════════════════════
   8 · COINS · PADS · CHECKPOINTS
   ══════════════════════════════════════════════════════════════ */
const coinGeo=new THREE.CylinderGeometry(.85,.85,.16,14);coinGeo.rotateX(Math.PI/2);
const coinMat=new THREE.MeshStandardMaterial({color:0xffc531,emissive:0xff9a10,emissiveIntensity:1.2,
  roughness:.28,metalness:.85,flatShading:true});
const coinIM=new THREE.InstancedMesh(coinGeo,coinMat,420);
coinIM.frustumCulled=false;coinIM.count=0;coinIM.instanceMatrix.setUsage(THREE.DynamicDrawUsage);scene.add(coinIM);

const padGeo=new THREE.PlaneGeometry(6.4,8.4,1,1);
const padTex=(function(){
  const cv=document.createElement('canvas');cv.width=128;cv.height=168;
  const c=cv.getContext('2d');
  c.fillStyle='rgba(8,18,28,0.82)';c.fillRect(0,0,128,168);
  c.strokeStyle='#3fe0ff';c.lineWidth=6;c.strokeRect(5,5,118,158);
  c.fillStyle='#7cf0ff';
  for(let i=0;i<3;i++){const y=120-i*40;c.beginPath();c.moveTo(24,y);c.lineTo(64,y-30);c.lineTo(104,y);
    c.lineTo(104,y+14);c.lineTo(64,y-16);c.lineTo(24,y+14);c.closePath();c.fill();}
  const t=new THREE.CanvasTexture(cv);t.colorSpace=THREE.SRGBColorSpace;return t;
})();
const padPool=[],activePads=[];
function getPad(){let m=padPool.pop();
  if(!m)m=new THREE.Mesh(padGeo,new THREE.MeshBasicMaterial({map:padTex,transparent:true,opacity:.9,
    depthWrite:false,side:THREE.DoubleSide,toneMapped:false}));
  m.rotation.x=-Math.PI/2;m.renderOrder=2;scene.add(m);activePads.push(m);return m;}
function releasePad(m){scene.remove(m);const i=activePads.indexOf(m);if(i>=0)activePads.splice(i,1);padPool.push(m);}
function releaseAllPads(){for(const o of worldObjects)if(o.t==='pad'&&o.mesh){releasePad(o.mesh);o.mesh=null;}}

const GATE_HALF=13.5;
let checkpoints=[],cpPool=[],cpCount=0;
function buildGate(){
  const g=new THREE.Group();
  const pg=new THREE.CylinderGeometry(.42,.72,11,6);
  const pm=new THREE.MeshStandardMaterial({color:0xf2f4f7,roughness:.6,metalness:.15,flatShading:true});
  const bm=new THREE.MeshBasicMaterial({color:0xff7a18,toneMapped:false});
  for(const s of[-1,1]){
    const py=new THREE.Mesh(pg,pm);py.position.set(s*GATE_HALF,5.5,0);py.castShadow=true;g.add(py);
    for(let b=0;b<3;b++){const bd=new THREE.Mesh(new THREE.BoxGeometry(1.5,.62,1.5),bm);
      bd.position.set(s*GATE_HALF,2.4+b*3.1,0);g.add(bd);}
    const fl=new THREE.Mesh(new THREE.PlaneGeometry(3.2,1.9),
      new THREE.MeshBasicMaterial({color:s<0?0xff7a18:0xffd23f,side:THREE.DoubleSide,toneMapped:false}));
    fl.position.set(s*(GATE_HALF+(s<0?-1.6:1.6)),10.6,0);g.add(fl);
  }
  const banner=new THREE.Mesh(new THREE.BoxGeometry(GATE_HALF*2,1.15,.28),
    new THREE.MeshStandardMaterial({color:0x14181d,roughness:.8,flatShading:true}));
  banner.position.y=11.4;banner.name='banner';g.add(banner);
  const strip=new THREE.Mesh(new THREE.BoxGeometry(GATE_HALF*2-.6,.32,.34),
    new THREE.MeshBasicMaterial({color:0x63e6ff,toneMapped:false}));
  strip.position.set(0,11.4,.03);g.add(strip);
  const ring=new THREE.Mesh(new THREE.TorusGeometry(2.1,.24,6,20),
    new THREE.MeshBasicMaterial({color:0xffd23f,toneMapped:false}));
  ring.position.set(0,8,0);g.add(ring);g.userData.ring=ring;
  const gr=new THREE.Mesh(new THREE.PlaneGeometry(GATE_HALF*2+2,7),
    new THREE.MeshBasicMaterial({color:0xff7a18,transparent:true,opacity:.26,depthWrite:false,
      toneMapped:false,blending:THREE.AdditiveBlending}));
  gr.rotation.x=-Math.PI/2;gr.position.y=.14;gr.renderOrder=2;g.add(gr);g.userData.ground=gr;
  return g;
}
function cpTex(txt){
  const cv=document.createElement('canvas');cv.width=512;cv.height=96;const c=cv.getContext('2d');
  c.fillStyle='#14181d';c.fillRect(0,0,512,96);
  c.fillStyle='#ff7a18';c.fillRect(0,0,10,96);c.fillRect(502,0,10,96);
  c.font='700 56px "Barlow Condensed", Impact, sans-serif';
  c.textAlign='center';c.textBaseline='middle';c.fillStyle='#fff';c.fillText(txt,256,52);
  const t=new THREE.CanvasTexture(cv);t.colorSpace=THREE.SRGBColorSpace;return t;
}
function getGate(){let g=cpPool.pop();if(!g)g=buildGate();scene.add(g);return g;}
function releaseGate(g){scene.remove(g);if(g.userData.tex){g.userData.tex.dispose();g.userData.tex=null;}cpPool.push(g);}
function releaseAllGates(){for(const cp of checkpoints)releaseGate(cp.mesh);checkpoints=[];}

function spawnCheckpoint(){
  const cp={},prev=checkpoints[checkpoints.length-1];
  let base;
  if(prev){
    const toCar=Math.atan2(car.pos.x-prev.x,car.pos.z-prev.z);
    base=prev.dir+angDiff(toCar,prev.dir)*0.5;
  }else base=car.heading;
  cp.dir=base+(Math.random()-.5)*1.45;
  const dist=prev?(135+Math.random()*80):195;
  if(prev){cp.x=prev.x+Math.sin(cp.dir)*dist;cp.z=prev.z+Math.cos(cp.dir)*dist;}
  else{cp.x=car.pos.x+Math.sin(cp.dir)*dist;cp.z=car.pos.z+Math.cos(cp.dir)*dist;}
  cp.y=groundHeight(cp.x,cp.z);
  let guard=0;
  while(cp.y<paramsAt(cp.z).water+2.6&&guard++<14){cp.x+=Math.sin(cp.dir)*42;cp.z+=Math.cos(cp.dir)*42;cp.y=groundHeight(cp.x,cp.z);}
  clearArea(cp.x,cp.z,GATE_HALF+9);
  cp.index=++cpCount;cp.taken=false;cp.radius=GATE_HALF+5;
  cp.mesh=getGate();
  cp.mesh.position.set(cp.x,cp.y,cp.z);
  cp.mesh.rotation.y=cp.dir+Math.PI;
  const banner=cp.mesh.getObjectByName('banner');
  if(banner){
    if(banner.material.map)banner.material.map.dispose();
    banner.material=banner.material.clone();
    banner.material.map=cpTex('CHECKPOINT '+cp.index);banner.material.needsUpdate=true;
    cp.mesh.userData.tex=banner.material.map;
  }
  checkpoints.push(cp);
}
function maintainCheckpoints(){
  for(let i=checkpoints.length-1;i>=0;i--){
    const cp=checkpoints[i];
    const dx=cp.x-car.pos.x,dz=cp.z-car.pos.z,d2=dx*dx+dz*dz;
    if((cp.taken&&d2>95*95)||d2>780*780){releaseGate(cp.mesh);checkpoints.splice(i,1);}
  }
  let guard=0;
  while(checkpoints.length<4&&guard++<10)spawnCheckpoint();
}
function nextCP(){for(let i=0;i<checkpoints.length;i++)if(!checkpoints[i].taken)return checkpoints[i];return null;}
function updateCoins(dt,time){
  let ci=0;const px=car.pos.x,pz=car.pos.z;
  for(let k=0;k<worldObjects.length;k++){
    const o=worldObjects[k];
    if(o.t!=='coin'||o.got||ci>=420)continue;
    const dx=o.x-px,dz=o.z-pz;
    if(dx*dx+dz*dz>260*260)continue;
    _v3.set(o.x,o.y+Math.sin(time*2.4+o.rot)*.28,o.z);
    _eul.set(0,time*2.1+o.rot,Math.PI/2);_q.setFromEuler(_eul);_sv.set(1,1,1);
    _m4.compose(_v3,_q,_sv);coinIM.setMatrixAt(ci++,_m4);
  }
  coinIM.count=ci;coinIM.instanceMatrix.needsUpdate=true;
}

/* ══════════════════════════════════════════════════════════════
   9 · CAR
   ══════════════════════════════════════════════════════════════ */
function buildCar(opt){
  const ghost=!!opt.ghost;
  const g=new THREE.Group(),body=new THREE.Group();g.add(body);g.userData.body=body;
  const M=(c,r,m,e)=>{
    const o={color:c,roughness:r,metalness:m,flatShading:true};
    if(e){o.emissive=e;o.emissiveIntensity=2.2;o.toneMapped=false;}
    if(ghost){o.transparent=true;o.opacity=opt.opacity||0.34;o.depthWrite=false;o.emissiveIntensity=e?0.6:0;}
    return new THREE.MeshStandardMaterial(o);
  };
  const paint=M(opt.paint,.34,.35),paintD=M(opt.paintD,.5,.25),trim=M(0x1a1d22,.75,.2),
        glass=ghost?M(0x8fd8ff,.1,.5):new THREE.MeshStandardMaterial({color:0x22303c,roughness:.08,metalness:.6,
          flatShading:true,transparent:true,opacity:.82}),
        chrome=M(0xd7dde4,.22,.9),light=M(0xfff6d8,.2,0,ghost?null:0xffedb0),
        tail=M(0xff2020,.3,0,ghost?null:0xff1a1a),white=M(0xffffff,.5,0);
  const B=(w,h,d,mat,x,y,z,rx,ry,rz)=>{
    const m=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),mat);m.position.set(x,y,z);
    if(rx||ry||rz)m.rotation.set(rx||0,ry||0,rz||0);
    if(!ghost){m.castShadow=true;m.receiveShadow=true;}
    body.add(m);return m;
  };
  B(1.86,.62,4.35,paint,0,.72,0);
  B(1.94,.34,3.10,paintD,0,.48,-.1);
  B(1.74,.40,1.15,paint,0,.80,1.90,-.14,0,0);
  B(1.80,.20,.55,trim,0,.56,2.42);
  B(1.62,.62,1.85,paint,0,1.32,-.18);
  B(1.44,.52,.95,glass,0,1.36,.62,-.30,0,0);
  B(1.42,.44,.80,glass,0,1.36,-1.06,.34,0,0);
  B(.06,.40,1.55,glass,.82,1.34,-.25);B(.06,.40,1.55,glass,-.82,1.34,-.25);
  B(.46,.16,.62,trim,0,1.68,.34);
  B(.05,.85,.05,chrome,-.62,1.98,-.95);
  B(1.86,.10,.60,trim,0,1.62,-2.02);
  B(.14,.44,.34,chrome,.66,1.38,-2.0);B(.14,.44,.34,chrome,-.66,1.38,-2.0);
  B(.44,.20,.10,light,.60,.90,2.44);B(.44,.20,.10,light,-.60,.90,2.44);
  B(.14,.14,.10,light,.20,.90,2.44);B(.14,.14,.10,light,-.20,.90,2.44);
  B(.52,.20,.10,tail,.58,1.00,-2.20);B(.52,.20,.10,tail,-.58,1.00,-2.20);
  if(!ghost)B(.44,.02,4.30,white,0,1.04,.02);
  B(1.98,.18,.9,trim,0,.42,1.35);B(1.98,.18,.9,trim,0,.42,-1.35);
  B(.12,.34,.06,trim,.72,.34,-2.28);B(.12,.34,.06,trim,-.72,.34,-2.28);
  B(.9,.14,.20,trim,0,1.70,.60);
  for(let i=-1;i<=1;i++)B(.20,.20,.10,light,i*.30,1.72,.70);

  const WR=0.47,WB=2.78,TR=1.62;
  const wg=new THREE.CylinderGeometry(WR,WR,.36,12);wg.rotateZ(Math.PI/2);
  const rg=new THREE.CylinderGeometry(.26,.26,.38,8);rg.rotateZ(Math.PI/2);
  const tireM=ghost?M(0x8fd8ff,.9,0):new THREE.MeshStandardMaterial({color:0x16181c,roughness:.95,flatShading:true});
  const rimM=ghost?M(0xbfeaff,.4,.6):new THREE.MeshStandardMaterial({color:0xe8c766,roughness:.35,metalness:.75,flatShading:true});
  const wheels=[];
  for(let i=0;i<4;i++){
    const front=i<2,left=(i%2)===0;
    const pivot=new THREE.Group();
    pivot.position.set(left?-TR/2:TR/2,WR,front?WB/2:-WB/2);
    const spin=new THREE.Group();
    const tire=new THREE.Mesh(wg,tireM);if(!ghost)tire.castShadow=true;spin.add(tire);
    spin.add(new THREE.Mesh(rg,rimM));
    pivot.add(spin);body.add(pivot);
    wheels.push({pivot,spin,front,left});
  }
  g.userData.wheels=wheels;g.userData.WR=WR;g.userData.WB=WB;
  if(!ghost){
    const bl=new THREE.SpotLight(0xfff0cc,0,95,.55,.5,1.2);
    bl.position.set(0,1,2.2);bl.target.position.set(0,0,30);body.add(bl);body.add(bl.target);
    g.userData.beam=bl;
  }
  return g;
}
const carGroup=buildCar({paint:0xd8232a,paintD:0x8e1218});scene.add(carGroup);
const wheels=carGroup.userData.wheels,WHEEL_R=carGroup.userData.WR,WHEELBASE=carGroup.userData.WB;
const beamL=carGroup.userData.beam;
const ghostCar=buildCar({paint:0x63e6ff,paintD:0x2a7fa8,ghost:true,opacity:0.30});
ghostCar.visible=false;scene.add(ghostCar);

const car={pos:new THREE.Vector3(),vel:new THREE.Vector3(),heading:0,steer:0,vy:0,onGround:true,
  vForward:0,vLat:0,up:new THREE.Vector3(0,1,0),boosting:false,boost:0,hp:100,airTime:0,bestAir:0,
  driftAmount:0,topSpeed:0,spinOut:0,handbrake:false};
const CAR_RIDE=0.02,GRAV=17.5;
const ENGINE_P=17.5,BRAKE_P=36,REVERSE_P=10,MAX_SPEED=44,MAX_SPEED_BOOST=69;
const LAT_GRIP=21,LAT_GRIP_HB=8.5;
const _fwd=new THREE.Vector3(),_right=new THREE.Vector3(),_upv=new THREE.Vector3();
const _tUp=new THREE.Vector3(),_tFwd=new THREE.Vector3(),_tRight=new THREE.Vector3();
const _basis=new THREE.Matrix4(),_qT=new THREE.Quaternion();
let lastGroundY=0,pitchS=0,rollS=0;
function resetCar(){
  car.pos.set(0,groundHeight(0,0)+0.6,0);car.vel.set(0,0,0);
  car.heading=0;car.steer=0;car.vy=0;car.onGround=true;car.vForward=0;car.vLat=0;
  car.hp=100;car.boost=35;car.airTime=0;car.bestAir=0;car.driftAmount=0;car.topSpeed=0;car.spinOut=0;
  groundNormal(car.pos.x,car.pos.z,car.up);
  lastGroundY=car.pos.y;pitchS=0;rollS=0;
  buildCarQuat();carGroup.quaternion.copy(_qT);carGroup.position.copy(car.pos);
  trailReset();
}
function buildCarQuat(){
  const h=car.heading;
  _tFwd.set(Math.sin(h),0,Math.cos(h));
  _tRight.set(Math.cos(h),0,-Math.sin(h));
  _tUp.copy(car.up);
  if(car.onGround){_tUp.applyAxisAngle(_tFwd,rollS);_tFwd.applyAxisAngle(_tRight,pitchS);}
  _tUp.normalize();
  _tRight.crossVectors(_tUp,_tFwd).normalize();
  _tFwd.crossVectors(_tRight,_tUp).normalize();
  _basis.makeBasis(_tRight,_tUp,_tFwd);
  _qT.setFromRotationMatrix(_basis);
}

/* ══════════════════════════════════════════════════════════════
   10 · PARTICLES + SKID TRAIL
   ══════════════════════════════════════════════════════════════ */
const PMAX=1000;
const pPos=new Float32Array(PMAX*3),pCol=new Float32Array(PMAX*3),pAl=new Float32Array(PMAX),pSz=new Float32Array(PMAX);
const pVel=new Float32Array(PMAX*3),pLife=new Float32Array(PMAX),pMaxL=new Float32Array(PMAX),pDrag=new Float32Array(PMAX);
let pHead=0,partScale=1;
const pGeo=new THREE.BufferGeometry();
pGeo.setAttribute('position',new THREE.BufferAttribute(pPos,3).setUsage(THREE.DynamicDrawUsage));
pGeo.setAttribute('aColor',new THREE.BufferAttribute(pCol,3).setUsage(THREE.DynamicDrawUsage));
pGeo.setAttribute('aAlpha',new THREE.BufferAttribute(pAl,1).setUsage(THREE.DynamicDrawUsage));
pGeo.setAttribute('aSize',new THREE.BufferAttribute(pSz,1).setUsage(THREE.DynamicDrawUsage));
const pMat=new THREE.ShaderMaterial({uniforms:{uScale:{value:400}},
  vertexShader:'attribute vec3 aColor;attribute float aAlpha;attribute float aSize;uniform float uScale;'+
   'varying vec3 vC;varying float vA;void main(){vC=aColor;vA=aAlpha;vec4 mv=modelViewMatrix*vec4(position,1.0);'+
   'gl_PointSize=aSize*uScale/max(-mv.z,0.6);gl_Position=projectionMatrix*mv;}',
  fragmentShader:'varying vec3 vC;varying float vA;void main(){vec2 d=gl_PointCoord-0.5;float r=dot(d,d);'+
   'if(r>0.25)discard;float a=vA*smoothstep(0.25,0.02,r);gl_FragColor=vec4(vC,a);}',
  transparent:true,depthWrite:false});
const points=new THREE.Points(pGeo,pMat);points.frustumCulled=false;scene.add(points);
function spawnParticle(x,y,z,vx,vy,vz,r,g,b,size,life,drag){
  if(partScale<=0)return;
  if(Math.random()>partScale)return;
  const i=pHead;pHead=(pHead+1)%PMAX;
  pPos[i*3]=x;pPos[i*3+1]=y;pPos[i*3+2]=z;
  pVel[i*3]=vx;pVel[i*3+1]=vy;pVel[i*3+2]=vz;
  pCol[i*3]=r;pCol[i*3+1]=g;pCol[i*3+2]=b;
  pSz[i]=size;pLife[i]=life;pMaxL[i]=life;pDrag[i]=drag;pAl[i]=1;
}
function updateParticles(dt){
  for(let i=0;i<PMAX;i++){
    if(pLife[i]<=0){pAl[i]=0;continue;}
    pLife[i]-=dt;
    const d=Math.exp(-pDrag[i]*dt);
    pVel[i*3]*=d;pVel[i*3+2]*=d;pVel[i*3+1]=pVel[i*3+1]*d-6.5*dt;
    pPos[i*3]+=pVel[i*3]*dt;pPos[i*3+1]+=pVel[i*3+1]*dt;pPos[i*3+2]+=pVel[i*3+2]*dt;
    const gy=groundHeight(pPos[i*3],pPos[i*3+2]);
    if(pPos[i*3+1]<gy){pPos[i*3+1]=gy;pVel[i*3+1]*=-.24;pVel[i*3]*=.72;pVel[i*3+2]*=.72;}
    const t=clamp(pLife[i]/pMaxL[i],0,1);pAl[i]=t*t;
  }
  pGeo.attributes.position.needsUpdate=true;pGeo.attributes.aColor.needsUpdate=true;
  pGeo.attributes.aAlpha.needsUpdate=true;pGeo.attributes.aSize.needsUpdate=true;
}
function burst(x,y,z,n,r,g,b,spd,size,life){
  for(let i=0;i<n;i++){
    const a=Math.random()*TAU,e=Math.random()*Math.PI*.5,s=spd*(.35+Math.random()*.9);
    spawnParticle(x,y,z,Math.cos(a)*Math.cos(e)*s,Math.sin(e)*s*1.15,Math.sin(a)*Math.cos(e)*s,
      r+Math.random()*.06,g+Math.random()*.06,b+Math.random()*.05,size*(.6+Math.random()*.8),
      life*(.6+Math.random()*.8),1.6);
  }
}
const TRAIL_N=240,TRAIL_LIFE=13;
const trailPts=[[],[]];
const trailGeo=new THREE.BufferGeometry();
const tvCount=TRAIL_N*2*2;
const tPos=new Float32Array(tvCount*3),tAl=new Float32Array(tvCount),tIdx=new Uint16Array(TRAIL_N*2*6);
trailGeo.setAttribute('position',new THREE.BufferAttribute(tPos,3).setUsage(THREE.DynamicDrawUsage));
trailGeo.setAttribute('aAlpha',new THREE.BufferAttribute(tAl,1).setUsage(THREE.DynamicDrawUsage));
trailGeo.setIndex(new THREE.BufferAttribute(tIdx,1));
const trailMat=new THREE.ShaderMaterial({uniforms:{uColor:{value:new THREE.Color(0x241d15)}},
  vertexShader:'attribute float aAlpha;varying float vA;void main(){vA=aAlpha;gl_Position=projectionMatrix*modelViewMatrix*vec4(position,1.0);}',
  fragmentShader:'uniform vec3 uColor;varying float vA;void main(){if(vA<0.012)discard;gl_FragColor=vec4(uColor,vA*0.62);}',
  transparent:true,depthWrite:false,side:THREE.DoubleSide});
const trailMesh=new THREE.Mesh(trailGeo,trailMat);trailMesh.frustumCulled=false;trailMesh.renderOrder=1;scene.add(trailMesh);
let trailAccum=0;
function trailReset(){trailPts[0].length=0;trailPts[1].length=0;rebuildTrail(0);}
function pushTrail(t){
  for(let s=0;s<2;s++){
    const wp=new THREE.Vector3();wheels[2+s].pivot.getWorldPosition(wp);
    const a=trailPts[s];
    a.push({x:wp.x,y:wp.y+.05,z:wp.z,born:t,a:car.driftAmount});
    if(a.length>TRAIL_N)a.shift();
  }
}
function rebuildTrail(time){
  if(!S.trail||!PQ().trail){trailMesh.visible=false;return;}
  trailMesh.visible=true;
  let vi=0,ii=0;
  for(let s=0;s<2;s++){
    const arr=trailPts[s];
    for(let i=0;i<TRAIL_N;i++){
      const p=arr[i];let a=0,dx=0,dz=1;
      if(p){
        const nx=arr[i+1]||p,pv=arr[i-1]||p;
        dx=nx.x-pv.x;dz=nx.z-pv.z;const l=Math.hypot(dx,dz)||1;dx/=l;dz/=l;
        a=p.a*clamp(1-(time-p.born)/TRAIL_LIFE,0,1);
      }
      const w=.20;
      tPos[vi*3]=p?p.x-dz*w:0;tPos[vi*3+1]=p?p.y:0;tPos[vi*3+2]=p?p.z+dx*w:0;tAl[vi]=a;vi++;
      tPos[vi*3]=p?p.x+dz*w:0;tPos[vi*3+1]=p?p.y:0;tPos[vi*3+2]=p?p.z-dx*w:0;tAl[vi]=a;vi++;
    }
    for(let i=0;i<TRAIL_N-1;i++){
      const a0=s*TRAIL_N*2+i*2;
      tIdx[ii++]=a0;tIdx[ii++]=a0+1;tIdx[ii++]=a0+2;
      tIdx[ii++]=a0+1;tIdx[ii++]=a0+3;tIdx[ii++]=a0+2;
    }
  }
  trailGeo.attributes.position.needsUpdate=true;trailGeo.attributes.aAlpha.needsUpdate=true;
  trailGeo.index.needsUpdate=true;
}

/* ══════════════════════════════════════════════════════════════
   11 · AUDIO
   ══════════════════════════════════════════════════════════════ */
const SFX={ctx:null,ok:false,master:null,engBus:null,sfxBus:null,
  init(){
    if(this.ctx)return;
    const AC=window.AudioContext||window.webkitAudioContext;if(!AC)return;
    try{this.ctx=new AC();}catch(e){return;}
    this.ok=true;
    const comp=this.ctx.createDynamicsCompressor();
    comp.threshold.value=-14;comp.ratio.value=6;comp.connect(this.ctx.destination);
    this.master=this.ctx.createGain();this.master.gain.value=S.vol/100*0.8;this.master.connect(comp);
    this.engBus=this.ctx.createGain();this.engBus.gain.value=S.eng/100;this.engBus.connect(this.master);
    this.sfxBus=this.ctx.createGain();this.sfxBus.gain.value=S.sfx/100;this.sfxBus.connect(this.master);

    this.engGain=this.ctx.createGain();this.engGain.gain.value=0;
    this.engFilt=this.ctx.createBiquadFilter();this.engFilt.type='lowpass';this.engFilt.frequency.value=900;
    this.o1=this.ctx.createOscillator();this.o1.type='sawtooth';this.o1.frequency.value=60;
    this.o2=this.ctx.createOscillator();this.o2.type='square';this.o2.frequency.value=30;
    this.o2g=this.ctx.createGain();this.o2g.gain.value=.45;
    this.o1.connect(this.engFilt);this.o2.connect(this.o2g);this.o2g.connect(this.engFilt);
    this.engFilt.connect(this.engGain);this.engGain.connect(this.engBus);
    this.o1.start();this.o2.start();

    const len=this.ctx.sampleRate*2,buf=this.ctx.createBuffer(1,len,this.ctx.sampleRate),d=buf.getChannelData(0);
    for(let i=0;i<len;i++)d[i]=Math.random()*2-1;
    this.nsrc=this.ctx.createBufferSource();this.nsrc.buffer=buf;this.nsrc.loop=true;
    this.nfilt=this.ctx.createBiquadFilter();this.nfilt.type='bandpass';this.nfilt.frequency.value=1100;this.nfilt.Q.value=.65;
    this.ngain=this.ctx.createGain();this.ngain.gain.value=0;
    this.nsrc.connect(this.nfilt);this.nfilt.connect(this.ngain);this.ngain.connect(this.engBus);this.nsrc.start();
    this.skGain=this.ctx.createGain();this.skGain.gain.value=0;
    this.skFilt=this.ctx.createBiquadFilter();this.skFilt.type='highpass';this.skFilt.frequency.value=2400;
    const s2=this.ctx.createBufferSource();s2.buffer=buf;s2.loop=true;s2.connect(this.skFilt);
    this.skFilt.connect(this.skGain);this.skGain.connect(this.engBus);s2.start();
  },
  resume(){if(this.ctx&&this.ctx.state==='suspended')this.ctx.resume();},
  setVol(){if(!this.ok)return;
    this.master.gain.value=muted?0:S.vol/100*0.8;
    this.engBus.gain.value=S.eng/100;this.sfxBus.gain.value=S.sfx/100;},
  engine(rpm,load,spd,alive){
    if(!this.ok)return;const t=this.ctx.currentTime;
    const f=42+rpm*190;
    this.o1.frequency.setTargetAtTime(f,t,.05);
    this.o2.frequency.setTargetAtTime(f*.5,t,.05);
    this.engFilt.frequency.setTargetAtTime(420+rpm*2100+spd*22,t,.06);
    this.engGain.gain.setTargetAtTime(alive?(.045+load*.075+rpm*.045):0,t,.10);
    this.ngain.gain.setTargetAtTime(alive?clamp(spd*.0042,0,.10):0,t,.12);
    this.skGain.gain.setTargetAtTime(alive?car.driftAmount*.055:0,t,.06);
  },
  blip(freq,dur,type,vol,slide){
    if(!this.ok)return;const t=this.ctx.currentTime;
    const o=this.ctx.createOscillator(),g=this.ctx.createGain();
    o.type=type||'sine';o.frequency.setValueAtTime(freq,t);
    if(slide)o.frequency.exponentialRampToValueAtTime(Math.max(40,slide),t+dur);
    g.gain.setValueAtTime(0,t);g.gain.linearRampToValueAtTime(vol||.22,t+.012);
    g.gain.exponentialRampToValueAtTime(.0001,t+dur);
    o.connect(g);g.connect(this.sfxBus);o.start(t);o.stop(t+dur+.03);
  },
  nh(dur,freq,vol){
    if(!this.ok)return;const t=this.ctx.currentTime,sr=this.ctx.sampleRate;
    const n=Math.max(8,Math.floor(sr*dur)),b=this.ctx.createBuffer(1,n,sr),d=b.getChannelData(0);
    for(let i=0;i<n;i++)d[i]=(Math.random()*2-1)*Math.pow(1-i/n,2);
    const s=this.ctx.createBufferSource();s.buffer=b;
    const f=this.ctx.createBiquadFilter();f.type='lowpass';f.frequency.setValueAtTime(freq,t);
    f.frequency.exponentialRampToValueAtTime(180,t+dur);
    const g=this.ctx.createGain();g.gain.setValueAtTime(vol,t);g.gain.exponentialRampToValueAtTime(.0001,t+dur);
    s.connect(f);f.connect(g);g.connect(this.sfxBus);s.start(t);
  },
  coin(){this.blip(1180,.11,'triangle',.16,1760);},
  crash(v){this.nh(.42,1400+v*220,clamp(.25+v*.012,.2,.7));this.blip(120,.3,'sawtooth',.16,44);},
  cp(){this.blip(660,.14,'square',.13);setTimeout(()=>this.blip(990,.2,'square',.13),95);},
  boostS(){this.nh(.55,3000,.2);this.blip(300,.5,'sawtooth',.1,1200);},
  land(v){this.nh(.2,700+v*22,clamp(.1+v*.008,.08,.4));},
  tick(){this.blip(880,.06,'sine',.1);}
};
let muted=false;

/* ══════════════════════════════════════════════════════════════
   12 · POST PROCESSING (lazy, non-blocking)
   ══════════════════════════════════════════════════════════════ */
let composer=null,bloomPass=null,ppTried=false;
function tryPost(){
  if(ppTried||composer)return;ppTried=true;
  const to=new Promise((_,rej)=>setTimeout(()=>rej(new Error('timeout')),6000));
  Promise.race([Promise.all([
      import('three/addons/postprocessing/EffectComposer.js'),
      import('three/addons/postprocessing/RenderPass.js'),
      import('three/addons/postprocessing/UnrealBloomPass.js'),
      import('three/addons/postprocessing/OutputPass.js')]),to])
    .then(([{EffectComposer},{RenderPass},{UnrealBloomPass},{OutputPass}])=>{
      composer=new EffectComposer(renderer);
      composer.addPass(new RenderPass(scene,camera));
      bloomPass=new UnrealBloomPass(new THREE.Vector2(innerWidth,innerHeight),.42,.62,.84);
      composer.addPass(bloomPass);
      composer.addPass(new OutputPass());
      applyGraphics(true);
    }).catch(()=>{composer=null;});
}

/* ══════════════════════════════════════════════════════════════
   13 · INPUT
   ══════════════════════════════════════════════════════════════ */
const keys={up:false,down:false,left:false,right:false,hand:false,boost:false};
const KEYMAP={KeyW:'up',ArrowUp:'up',KeyS:'down',ArrowDown:'down',KeyA:'left',ArrowLeft:'left',
  KeyD:'right',ArrowRight:'right',Space:'hand',ShiftLeft:'boost',ShiftRight:'boost'};
addEventListener('keydown',e=>{
  if(e.target&&/INPUT|TEXTAREA/.test(e.target.tagName))return;
  if(KEYMAP[e.code]){keys[KEYMAP[e.code]]=true;e.preventDefault();SFX.resume();}
  if(e.code==='Escape'){
    if(state==='playing')pauseGame();
    else if(state==='paused')resumeGame();
    else if(!$('settings').classList.contains('hidden'))closeSettings();
    else if(!$('mapSelect').classList.contains('hidden'))showScreen('mainMenu');
  }
  if(e.code==='KeyC'&&(state==='playing'||state==='paused')){S.cam=(S.cam+1)%3;syncCamSeg();saveSettings();toast('CAM · '+(['CHASE','HOOD','CINEMATIC'][S.cam]));}
  if(e.code==='KeyF')toggleFullscreen();
  if(e.code==='KeyM')toggleMute();
  if(e.code==='KeyG'&&state==='playing'){camModeGhost=!camModeGhost;toast(camModeGhost?'GHOST CAM ON':'GHOST CAM OFF');}
});
addEventListener('keyup',e=>{if(KEYMAP[e.code]){keys[KEYMAP[e.code]]=false;e.preventDefault();}});
addEventListener('blur',()=>{for(const k in keys)keys[k]=false;});

const isTouch=('ontouchstart'in window)||navigator.maxTouchPoints>0;
if(isTouch)$('touch').classList.add('on');
function bindT(id,key){
  const el=$(id);if(!el)return;
  const on=e=>{e.preventDefault();keys[key]=true;el.classList.add('act');SFX.init();SFX.resume();};
  const off=e=>{e.preventDefault();keys[key]=false;el.classList.remove('act');};
  el.addEventListener('touchstart',on,{passive:false});el.addEventListener('touchend',off,{passive:false});
  el.addEventListener('touchcancel',off,{passive:false});
  el.addEventListener('mousedown',on);el.addEventListener('mouseup',off);el.addEventListener('mouseleave',off);
}
bindT('tL','left');bindT('tR','right');bindT('tGas','up');bindT('tBrk','down');bindT('tNos','boost');bindT('tHnd','hand');

/* fullscreen */
function toggleFullscreen(){
  const d=document;
  try{
    if(!d.fullscreenElement&&!d.webkitFullscreenElement){
      const el=d.documentElement;
      (el.requestFullscreen||el.webkitRequestFullscreen||el.msRequestFullscreen).call(el);
    }else{(d.exitFullscreen||d.webkitExitFullscreen||d.msExitFullscreen).call(d);}
  }catch(e){}
}
function syncFS(){
  const on=!!(document.fullscreenElement||document.webkitFullscreenElement);
  $('btnFull').textContent=on?'🗗':'⛶';
}
document.addEventListener('fullscreenchange',syncFS);
document.addEventListener('webkitfullscreenchange',syncFS);
function toggleMute(){muted=!muted;$('btnSound').textContent=muted?'🔇':'🔊';SFX.setVol();}

/* ══════════════════════════════════════════════════════════════
   14 · GHOST SYSTEM
   ══════════════════════════════════════════════════════════════ */
const GH_DT=0.05,GH_MAX=4200;
let ghRec=null,ghPlay=null,ghDeltaEl=null;
function i16ToB64(arr){
  const bytes=new Uint8Array(arr.buffer,arr.byteOffset,arr.byteLength);
  let s='';const CH=0x8000;
  for(let i=0;i<bytes.length;i+=CH)s+=String.fromCharCode.apply(null,bytes.subarray(i,i+CH));
  return btoa(s);
}
function b64ToI16(str){
  const s=atob(str),b=new Uint8Array(s.length);
  for(let i=0;i<s.length;i++)b[i]=s.charCodeAt(i);
  return new Int16Array(b.buffer);
}
function ghostKey(id){return 'rr.ghost.'+id;}
function loadGhost(id){
  try{
    const raw=localStorage.getItem(ghostKey(id));if(!raw)return null;
    const o=JSON.parse(raw);const arr=b64ToI16(o.d);
    const n=arr.length/4,samples=new Float32Array(n*4);
    for(let i=0;i<n;i++){samples[i*4]=arr[i*4]/10;samples[i*4+1]=arr[i*4+1]/20;
      samples[i*4+2]=arr[i*4+2]/10;samples[i*4+3]=arr[i*4+3]/1000;}
    return {n,samples,score:o.s||0,dist:o.dist||0,cps:o.c||0};
  }catch(e){return null;}
}
function saveGhost(id,score,dist,cps){
  if(!ghRec||ghRec.n<10)return false;
  try{
    const n=Math.min(ghRec.n,GH_MAX),out=new Int16Array(n*4);
    for(let i=0;i<n;i++){
      out[i*4]=clamp(Math.round(ghRec.buf[i*4]*10),-32760,32760);
      out[i*4+1]=clamp(Math.round(ghRec.buf[i*4+1]*20),-32760,32760);
      out[i*4+2]=clamp(Math.round(ghRec.buf[i*4+2]*10),-32760,32760);
      out[i*4+3]=clamp(Math.round(ghRec.buf[i*4+3]*1000),-32760,32760);
    }
    localStorage.setItem(ghostKey(id),JSON.stringify({s:Math.floor(score),dist:Math.floor(dist),c:cps,d:i16ToB64(out)}));
    return true;
  }catch(e){return false;}
}
function startRecord(){ghRec={n:0,cap:GH_MAX,buf:new Float32Array(GH_MAX*4),t:0};}
function recordSample(dt){
  if(!ghRec)return;
  ghRec.t+=dt;
  if(ghRec.t<GH_DT)return;
  ghRec.t=0;
  if(ghRec.n>=ghRec.cap)return;
  const i=ghRec.n*4;
  ghRec.buf[i]=car.pos.x;ghRec.buf[i+1]=car.pos.y;ghRec.buf[i+2]=car.pos.z;ghRec.buf[i+3]=car.heading;
  ghRec.n++;
}
function ghostSample(g,t,out){
  const f=t/GH_DT,i0=Math.floor(f);
  if(i0>=g.n-1){const i=(g.n-1)*4;out.set(g.samples[i],g.samples[i+1],g.samples[i+2],g.samples[i+3]);return false;}
  if(i0<0){out.set(g.samples[0],g.samples[1],g.samples[2],g.samples[3]);return true;}
  const a=i0*4,b=a+4,w=f-i0;
  let h0=g.samples[a+3],h1=g.samples[b+3];
  h1=h0+angDiff(h1,h0);
  out.set(lerp(g.samples[a],g.samples[b],w),lerp(g.samples[a+1],g.samples[b+1],w),
          lerp(g.samples[a+2],g.samples[b+2],w),h0+(h1-h0)*w);
  return true;
}
const _gh=new Float32Array(4),_gh2=new Float32Array(4);
let camModeGhost=false;
function updateGhost(dt){
  if(!ghPlay){ghostCar.visible=false;return;}
  const t=runTime;
  const alive=ghostSample(ghPlay,t,_gh);
  if(!alive&&t>ghPlay.n*GH_DT+2){ghostCar.visible=false;$('ghostDelta').style.display='none';return;}
  ghostCar.visible=true;
  ghostCar.position.set(_gh[0],_gh[1],_gh[2]);
  groundNormal(_gh[0],_gh[2],_tUp);
  _tFwd.set(Math.sin(_gh[3]),0,Math.cos(_gh[3]));
  _tRight.crossVectors(_tUp,_tFwd).normalize();
  _tFwd.crossVectors(_tRight,_tUp).normalize();
  _basis.makeBasis(_tRight,_tUp,_tFwd);
  ghostCar.quaternion.setFromRotationMatrix(_basis);
  const gs=ghostCar.userData.wheels;
  for(let i=0;i<4;i++)gs[i].spin.rotation.x-=dt*14;
  // delta vs player distance
  ghostSample(ghPlay,Math.max(0,t-GH_DT),_gh2);
  const gd=Math.hypot(_gh[0]-car.pos.x,_gh[2]-car.pos.z);
  const ahead=((_gh[0]-car.pos.x)*Math.sin(car.heading)+(_gh[2]-car.pos.z)*Math.cos(car.heading))>0;
  const el=$('ghostDelta');
  el.style.display='block';
  el.textContent='GHOST '+(ahead?'+':'−')+Math.round(gd)+'m';
  el.style.color=ahead?'#63e6ff':'#ff6b6b';
  el.style.borderLeftColor=ahead?'#63e6ff':'#ff6b6b';
}

/* ══════════════════════════════════════════════════════════════
   15 · HUD PAINTING
   ══════════════════════════════════════════════════════════════ */
const hudEl=$('hud'),vigEl=$('vig'),boostFxEl=$('boostfx'),dmgFxEl=$('damagefx');
const miniCtx=$('mini').getContext('2d'),spdCtx=$('speedoCv').getContext('2d');
function toast(txt,color){
  const d=document.createElement('div');d.className='toastItem';d.textContent=txt;
  if(color)d.style.borderLeftColor=color;
  $('toast').appendChild(d);setTimeout(()=>d.remove(),1550);
}
function showCenter(big,small,dur){
  const cm=$('centerMsg');
  $('cmBig').textContent=big;$('cmSmall').textContent=small||'';
  cm.style.transition='none';cm.style.opacity='1';cm.style.transform='translate(-50%,-50%) scale(1.12)';
  requestAnimationFrame(()=>{cm.style.transition='opacity .35s,transform .35s';cm.style.transform='translate(-50%,-50%) scale(1)';});
  clearTimeout(showCenter._t);
  showCenter._t=setTimeout(()=>{cm.style.opacity='0';},dur||900);
}
function convSpeed(ms){
  const k=Math.abs(ms)*3.6;
  return S.units==='mph'?k*0.621371:k;
}
function drawSpeedo(kmh,bp){
  const c=spdCtx,Sz=336,cx=Sz/2,cy=Sz/2,R=Sz/2-16;
  c.clearRect(0,0,Sz,Sz);
  const A0=Math.PI*.75,A1=Math.PI*2.25;
  c.lineWidth=13;c.lineCap='butt';
  c.strokeStyle='rgba(255,255,255,.10)';c.beginPath();c.arc(cx,cy,R,A0,A1);c.stroke();
  const MAXK=S.units==='mph'?150:240;
  const frac=clamp(kmh/MAXK,0,1);
  const gr=c.createLinearGradient(0,Sz,Sz,0);
  gr.addColorStop(0,'#ffd23f');gr.addColorStop(.55,'#ff7a18');gr.addColorStop(1,'#ff2d55');
  c.strokeStyle=gr;c.beginPath();c.arc(cx,cy,R,A0,A0+(A1-A0)*frac);c.stroke();
  if(bp>.001){c.lineWidth=5;c.strokeStyle='rgba(99,230,255,'+(.35+bp*.6)+')';
    c.beginPath();c.arc(cx,cy,R-14,A0,A0+(A1-A0)*bp);c.stroke();}
  c.strokeStyle='rgba(255,255,255,.28)';c.lineWidth=2;
  const steps=S.units==='mph'?10:12;
  for(let i=0;i<=steps;i++){
    const a=A0+(A1-A0)*(i/steps),big=i%2===0,r0=R-24-(big?12:6),r1=R-24;
    c.beginPath();c.moveTo(cx+Math.cos(a)*r0,cy+Math.sin(a)*r0);c.lineTo(cx+Math.cos(a)*r1,cy+Math.sin(a)*r1);c.stroke();
    if(big){c.fillStyle='rgba(255,255,255,.4)';c.font='700 19px "Barlow Condensed",sans-serif';
      c.textAlign='center';c.textBaseline='middle';
      c.fillText(String(i*(S.units==='mph'?15:20)),cx+Math.cos(a)*(r0-18),cy+Math.sin(a)*(r0-18));}
  }
  const na=A0+(A1-A0)*frac;
  c.strokeStyle='#ff2d55';c.lineWidth=4;c.lineCap='round';
  c.beginPath();c.moveTo(cx-Math.cos(na)*16,cy-Math.sin(na)*16);
  c.lineTo(cx+Math.cos(na)*(R-34),cy+Math.sin(na)*(R-34));c.stroke();
  c.fillStyle='#12161c';c.beginPath();c.arc(cx,cy,13,0,TAU);c.fill();
  c.strokeStyle='#ff7a18';c.lineWidth=3;c.stroke();
}
function drawMinimap(){
  const c=miniCtx,Sz=256,cx=Sz/2,cy=Sz/2,R=Sz/2-6;
  c.clearRect(0,0,Sz,Sz);
  c.save();c.beginPath();c.arc(cx,cy,R,0,TAU);c.clip();
  c.fillStyle='rgba(8,12,17,.86)';c.fillRect(0,0,Sz,Sz);
  const sc=1.05;
  c.save();c.translate(cx,cy);c.rotate(-car.heading);
  c.strokeStyle='rgba(255,255,255,.06)';c.lineWidth=1;
  const gs=52,b0=Math.floor(car.pos.x/gs)*gs;
  for(let g=b0-312;g<=b0+312;g+=gs){
    c.beginPath();c.moveTo((g-car.pos.x)*sc,-320);c.lineTo((g-car.pos.x)*sc,320);c.stroke();
  }
  const b1=Math.floor(car.pos.z/gs)*gs;
  for(let g=b1-312;g<=b1+312;g+=gs){
    c.beginPath();c.moveTo(-320,-(g-car.pos.z)*sc);c.lineTo(320,-(g-car.pos.z)*sc);c.stroke();
  }
  for(let i=0;i<worldObjects.length;i++){
    const o=worldObjects[i];
    const dx=(o.x-car.pos.x)*sc,dz=-(o.z-car.pos.z)*sc;
    if(dx<-R||dx>R||dz<-R||dz>R)continue;
    if(o.t==='tree'){c.fillStyle='rgba(70,150,80,.5)';c.fillRect(dx-1.6,dz-1.6,3.2,3.2);}
    else if(o.t==='rock'){c.fillStyle='rgba(150,145,138,.45)';c.fillRect(dx-1.4,dz-1.4,2.8,2.8);}
    else if(o.t==='pad'){c.fillStyle='rgba(99,230,255,.85)';c.beginPath();c.arc(dx,dz,3.4,0,TAU);c.fill();}
  }
  const ncp=nextCP();
  for(let i=0;i<checkpoints.length;i++){
    const cp=checkpoints[i];if(cp.taken)continue;
    let dx=(cp.x-car.pos.x)*sc,dz=-(cp.z-car.pos.z)*sc;
    const d=Math.hypot(dx,dz);
    if(d>R-9){const a=Math.atan2(dz,dx);dx=Math.cos(a)*(R-9);dz=Math.sin(a)*(R-9);}
    const isN=cp===ncp,pulse=.6+.4*Math.sin(performance.now()*.006);
    c.fillStyle=isN?'rgba(255,122,24,'+(.75+pulse*.25)+')':'rgba(255,210,63,.4)';
    c.beginPath();c.arc(dx,dz,isN?8:5,0,TAU);c.fill();
    if(isN){c.strokeStyle='rgba(255,210,63,'+(.3+pulse*.5)+')';c.lineWidth=2.5;
      c.beginPath();c.arc(dx,dz,12+pulse*5,0,TAU);c.stroke();}
  }
  if(ghPlay&&ghostCar.visible){
    let dx=(ghostCar.position.x-car.pos.x)*sc,dz=-(ghostCar.position.z-car.pos.z)*sc;
    const d=Math.hypot(dx,dz);
    if(d>R-8){const a=Math.atan2(dz,dx);dx=Math.cos(a)*(R-8);dz=Math.sin(a)*(R-8);}
    c.fillStyle='rgba(99,230,255,.9)';c.beginPath();c.arc(dx,dz,5,0,TAU);c.fill();
  }
  c.fillStyle='#fff';c.beginPath();c.moveTo(0,-11);c.lineTo(7.5,8);c.lineTo(0,4);c.lineTo(-7.5,8);c.closePath();c.fill();
  c.strokeStyle='rgba(255,122,24,.9)';c.lineWidth=2;c.stroke();
  c.restore();
  c.strokeStyle='rgba(255,255,255,.18)';c.lineWidth=2;c.beginPath();c.arc(cx,cy,R,0,TAU);c.stroke();
  c.restore();
}
let hudAcc=0;
function tickHUD(){
  const kmh=convSpeed(car.vForward);
  $('kmh').textContent=Math.floor(kmh);
  $('kmhMini').innerHTML=Math.floor(kmh)+'<i>'+(S.units==='mph'?'MPH':'KM/H')+'</i>';
  $('unitLbl').textContent=S.units==='mph'?'MPH':'KM/H';
  $('gearNum').textContent=car.vForward<-.5?'R':(kmh<1?'N':String(clamp(Math.floor(kmh/(S.units==='mph'?24:38))+1,1,6)));
  $('scoreVal').textContent=Math.floor(score).toLocaleString();
  $('comboVal').textContent='×'+combo.toFixed(1);
  $('comboVal').style.opacity=combo>1.01?'1':'0';
  $('distVal').textContent=Math.floor(distance).toLocaleString();
  $('coinVal').textContent=coinsGot;
  $('airVal').textContent=car.airTime.toFixed(1);
  if(mode==='infinite'){
    $('timeLbl').textContent='Distance';
    $('timeBar').style.width=clamp((distance%1000)/10,'%');
    $('timeTxt').textContent=Math.floor(distance).toLocaleString()+' m';
    $('timeTxt').style.color='#63e6ff';
    $('cpLbl').textContent=paramsAt(car.pos.z).label;
  }else{
    $('timeLbl').textContent='Stage Time';
    $('timeBar').style.width=(clamp(timeLeft/Math.max(1,MAP.time),0,1)*100)+'%';
    $('timeBar').style.background=timeLeft<8?'linear-gradient(90deg,#ff2d55,#ff6b3d)':'linear-gradient(90deg,#ffd23f,#ff7a18)';
    $('timeTxt').textContent=timeLeft.toFixed(1);
    $('timeTxt').style.color=timeLeft<8?'#ff5a5a':'#fff';
    $('cpLbl').textContent='CP '+(cpCount+(nextCP()?1:0));
  }
  $('hpFill').style.width=clamp(car.hp,0,100)+'%';
  $('hpTxt').textContent=Math.floor(clamp(car.hp,0,100))+'%';
  $('boostFill').style.width=clamp(car.boost,0,100)+'%';
  $('boTxt').textContent=Math.floor(car.boost)+'%';
  const spdN=clamp(Math.abs(car.vForward)/MAX_SPEED_BOOST,0,1);
  vigEl.style.opacity=(.20+spdN*.62+(car.boosting?.3:0))*(S.shake/100).toFixed(3);
  boostFxEl.style.opacity=car.boosting?(.5+Math.random()*.25).toFixed(2):'0';
  $('airMeter').style.opacity=(!car.onGround&&car.airTime>.25)?'1':'0';
  $('airTime').textContent=car.airTime.toFixed(1)+'s';
  $('biomeTag').style.display=(mode==='infinite'&&state==='playing')?'block':'none';
  if(mode==='infinite')$('biomeTag').textContent=paramsAt(car.pos.z).label+' BIOME';
  if(!isSmallScreen())drawSpeedo(kmh,car.boost/100);
  drawMinimap();
}
function isSmallScreen(){return innerWidth<=760||matchMedia('(pointer:coarse)').matches;}

/* ══════════════════════════════════════════════════════════════
   16 · CAMERA
   ══════════════════════════════════════════════════════════════ */
let camYaw=0,camShake=0,previewOrbit=0;
const camPos=new THREE.Vector3(0,8,-14),camLook=new THREE.Vector3();
const _shk=new THREE.Vector3();
let previewTarget=new THREE.Vector3();
function updateCamera(dt){
  const spd=Math.hypot(car.vel.x,car.vel.z);
  let desired=car.heading;
  if(camModeGhost&&ghPlay&&ghostCar.visible)desired=Math.atan2(ghostCar.position.x-car.pos.x,ghostCar.position.z-car.pos.z);
  else if(S.cam===2){const n=nextCP();if(n)desired=Math.atan2(n.x-car.pos.x,n.z-car.pos.z);}
  camYaw+=angDiff(desired,camYaw)*(1-Math.exp(-(S.cam===1?15:4.8)*dt));
  const back=S.cam===1?0.55:(8.6+clamp(spd*.045,0,2.4));
  const up=S.cam===1?1.62:(3.55+clamp(spd*.014,0,.7));
  const tx=car.pos.x-Math.sin(camYaw)*back,tz=car.pos.z-Math.cos(camYaw)*back;
  let ty=car.pos.y+up;
  const gh=groundHeight(tx,tz)+1.9;if(ty<gh)ty=gh;
  camPos.x=damp(camPos.x,tx,13,dt);camPos.y=damp(camPos.y,ty,11,dt);camPos.z=damp(camPos.z,tz,13,dt);
  const la=S.cam===1?10:8.5;
  camLook.x=damp(camLook.x,car.pos.x+Math.sin(car.heading)*la,S.cam===1?26:9,dt);
  camLook.y=damp(camLook.y,car.pos.y+(S.cam===1?1.15:1.55),10,dt);
  camLook.z=damp(camLook.z,car.pos.z+Math.cos(car.heading)*la,S.cam===1?26:9,dt);
  camShake=Math.max(0,camShake-dt*2.4);
  const sk=camShake*(S.shake/100)*0.55;
  _shk.set(Math.random()-.5,Math.random()-.5,Math.random()-.5).multiplyScalar(sk);
  camera.position.copy(camPos).add(_shk);
  camera.lookAt(camLook);
  camera.rotation.z+=_shk.x*.03;
  const kmh=convSpeed(car.vForward);
  const baseFov=(S.cam===1?S.fov+13:S.fov);
  camera.fov=damp(camera.fov,baseFov+clamp(kmh/240*16,0,16)+(car.boosting?7:0),5.5,dt);
  camera.updateProjectionMatrix();
  sky.position.copy(camera.position);
  water.position.set(camera.position.x,paramsAt(car.pos.z).water,camera.position.z);
}
function updatePreviewCam(dt){
  previewOrbit+=dt*0.14;
  const r=46,h=15;
  const px=previewTarget.x+Math.sin(previewOrbit)*r,pz=previewTarget.z+Math.cos(previewOrbit)*r;
  let py=previewTarget.y+h;
  const gh=groundHeight(px,pz)+4;if(py<gh)py=gh;
  camPos.x=damp(camPos.x,px,2.4,dt);camPos.y=damp(camPos.y,py,2.4,dt);camPos.z=damp(camPos.z,pz,2.4,dt);
  camLook.x=damp(camLook.x,previewTarget.x,3,dt);
  camLook.y=damp(camLook.y,previewTarget.y+1.4,3,dt);
  camLook.z=damp(camLook.z,previewTarget.z,3,dt);
  camera.position.copy(camPos);camera.lookAt(camLook);
  camera.fov=damp(camera.fov,54,3,dt);camera.updateProjectionMatrix();
  sky.position.copy(camera.position);
  water.position.set(camera.position.x,paramsAt(previewTarget.z).water,camera.position.z);
}

/* ══════════════════════════════════════════════════════════════
   17 · GRAPHICS APPLICATION
   ══════════════════════════════════════════════════════════════ */
let dynScale=1,shadowOn=true,bloomOn=true;
function applyGraphics(rebuild){
  const q=PQ();
  shadowOn=q.shadow&&S.shadows;
  bloomOn=q.bloom&&S.bloom;
  renderer.shadowMap.enabled=shadowOn;
  sun.castShadow=shadowOn;
  if(sun.shadow.mapSize.x!==q.shSize){
    sun.shadow.mapSize.set(q.shSize,q.shSize);
    if(sun.shadow.map){sun.shadow.map.dispose();sun.shadow.map=null;}
  }
  trunkIM.castShadow=canopyIM.castShadow=rockIM.castShadow=bushIM.castShadow=shadowOn;
  partScale=(q.part*(S.part/100));
  pMat.uniforms.uScale.value=innerHeight*renderer.getPixelRatio()*0.5;
  if(bloomPass)bloomPass.enabled=bloomOn;
  if(rebuild){
    buildTerrain();
    releaseAllPads();releaseAllGates();
    cells.clear();invalidateParams();rebuildFlat();
    updateTerrain(car.pos.x,car.pos.z,true);
    refreshCells(car.pos.x,car.pos.z);rebuildProps();
    if(state==='playing'||state==='countdown'||state==='paused')maintainCheckpoints();
  }
  applyMapVisuals();
  onResize();
}
const _cc=new THREE.Color();
function srgb(a){return _cc.setRGB(a[0],a[1],a[2],THREE.SRGBColorSpace).clone();}
function applyMapVisuals(){
  const p=paramsAt(car.pos.z),q=PQ();
  const fogFar=p.span!==undefined?0:0;
  const far=q.span*(S.draw/100)*0.42*p.fogM;
  scene.fog.color.copy(srgb(p.cFog));
  scene.fog.near=far*0.30;scene.fog.far=far;
  skyU.uTop.value.copy(srgb(p.cSky0));
  skyU.uMid.value.copy(srgb(p.cSky1));
  skyU.uBot.value.copy(srgb(p.cSky2));
  skyU.uSun.value.copy(srgb(p.cSun));
  skyU.uHaze.value.copy(srgb(p.cFog));
  sunDirV.set(p.sunDir[0],p.sunDir[1],p.sunDir[2]).normalize();
  skyU.uSunDir.value.copy(sunDirV);
  sun.color.copy(srgb(p.cSun));sun.intensity=p.sunI;
  hemi.color.copy(srgb(p.hemiS));hemi.groundColor.copy(srgb(p.hemiG));hemi.intensity=p.hemiI;
  fillL.color.copy(srgb(p.cSky0));fillL.intensity=0.28+p.sunI*0.06;
  waterMat.color.copy(srgb(p.cW));
  water.position.y=p.water;
  waterMat.opacity=p.theme==='swamp'?0.94:0.86;
  trailMat.uniforms.uColor.value.copy(srgb(p.cA)).multiplyScalar(0.42);
  renderer.toneMappingExposure=p.theme==='volcanic'?1.22:1.06;
  if(bloomPass)bloomPass.threshold=p.theme==='volcanic'?0.62:0.84;
}
function onResize(){
  const w=innerWidth,h=innerHeight;
  camera.aspect=w/h;camera.updateProjectionMatrix();
  const q=PQ();
  const base=Math.min(window.devicePixelRatio||1,q.dpr);
  const pr=Math.max(0.4,base*(S.resScale/100)*dynScale);
  renderer.setPixelRatio(pr);
  renderer.setSize(w,h);
  if(composer)composer.setSize(w,h);
  if(bloomPass)bloomPass.resolution.set(w,h);
  pMat.uniforms.uScale.value=h*pr*0.5;
}
addEventListener('resize',onResize);

/* ══════════════════════════════════════════════════════════════
   18 · PHYSICS + INTERACTION
   ══════════════════════════════════════════════════════════════ */
let nearby=[];
function physics(dt){
  const alive=(state==='playing');
  const ctrl=alive?1:0;
  const p=paramsAt(car.pos.z);
  const throttle=(keys.up?1:0)*ctrl;
  const brakeIn=(keys.down?1:0)*ctrl;
  const steerIn=((keys.left?1:0)-(keys.right?1:0))*ctrl;
  const hb=!!keys.hand&&alive;car.handbrake=hb;
  const wantBoost=!!keys.boost&&alive&&car.boost>1&&throttle>0;
  car.boosting=wantBoost;
  if(wantBoost)car.boost=Math.max(0,car.boost-dt*26);
  else if(alive)car.boost=Math.min(100,car.boost+dt*1.6);

  _fwd.set(Math.sin(car.heading),0,Math.cos(car.heading));
  _right.set(Math.cos(car.heading),0,-Math.sin(car.heading));
  let vF=car.vel.dot(_fwd),vL=car.vel.dot(_right);

  const maxSpd=wantBoost?MAX_SPEED_BOOST:MAX_SPEED;
  let accel=0;
  if(throttle>0)accel+=ENGINE_P*(wantBoost?1.75:1)*(1-clamp(Math.abs(vF)/maxSpd,0,1));
  if(brakeIn>0){if(vF>0.6)accel-=BRAKE_P*(hb?1.25:1);else accel-=REVERSE_P*(1-clamp(-vF/14,0,1));}
  accel-=vF*0.30*p.drag;
  accel-=Math.sign(vF)*vF*vF*0.0042*p.drag;
  if(hb)accel-=Math.sign(vF)*7.5;

  const grade=(groundHeight(car.pos.x+_fwd.x*2.4,car.pos.z+_fwd.z*2.4)
             -groundHeight(car.pos.x-_fwd.x*2.4,car.pos.z-_fwd.z*2.4))/4.8;
  if(car.onGround)accel-=GRAV*grade*0.62;

  vF+=accel*dt;
  if(!throttle&&!brakeIn&&Math.abs(vF)<0.35)vF*=Math.exp(-6*dt);
  vF=clamp(vF,-16,maxSpd*1.12);

  const assistMul=S.assist?1.0:0.86;
  const latGrip=(hb?LAT_GRIP_HB:LAT_GRIP)*p.grip*assistMul*(car.onGround?1:0.25)*(wantBoost?1.06:1);
  const minR=Math.max(vF*vF/Math.max(0.5,latGrip),4.4);
  const maxSteer=Math.atan(WHEELBASE/minR);
  car.steer=damp(car.steer,steerIn*maxSteer,11,dt);
  let yawRate=(vF/WHEELBASE)*Math.tan(car.steer);
  if(hb)yawRate*=1.62;
  if(!car.onGround)yawRate*=0.30;
  if(state==='over')yawRate=car.spinOut;
  if(S.assist&&car.onGround&&!hb&&Math.abs(steerIn)<0.1)yawRate*=0.985;
  car.heading+=yawRate*dt;

  const latDecay=hb?2.4:(car.onGround?9.6*p.grip:0.55);
  vL*=Math.exp(-latDecay*dt);

  _fwd.set(Math.sin(car.heading),0,Math.cos(car.heading));
  _right.set(Math.cos(car.heading),0,-Math.sin(car.heading));
  car.vel.copy(_fwd).multiplyScalar(vF).addScaledVector(_right,vL);
  car.vForward=vF;car.vLat=vL;

  car.pos.x+=car.vel.x*dt;car.pos.z+=car.vel.z*dt;

  const gy=groundHeight(car.pos.x,car.pos.z)+CAR_RIDE;
  if(car.onGround){
    const slopeVy=(gy-lastGroundY)/Math.max(dt,1e-5);
    car.vy=clamp(slopeVy,-45,45);
    car.pos.y=gy;lastGroundY=gy;
    const la=0.10;
    const fx=car.pos.x+car.vel.x*la,fz=car.pos.z+car.vel.z*la;
    const fgy=groundHeight(fx,fz)+CAR_RIDE;
    const fy=car.pos.y+car.vy*la-0.5*GRAV*la*la;
    if(fy>fgy+0.03)car.onGround=false;
  }else{
    car.vy-=GRAV*dt;car.pos.y+=car.vy*dt;
    if(car.pos.y<=gy){
      car.pos.y=gy;const impact=-car.vy;car.vy=0;car.onGround=true;lastGroundY=gy;
      if(impact>4){
        camShake=Math.min(1.1,impact*0.045);SFX.land(impact);
        const n=Math.min(46,Math.floor(impact*3.2));
        const cc=paramsAt(car.pos.z).cA;
        for(let i=0;i<n;i++){
          const a=Math.random()*TAU,r=Math.random()*1.8;
          spawnParticle(car.pos.x+Math.cos(a)*r,car.pos.y+.25,car.pos.z+Math.sin(a)*r,
            Math.cos(a)*(2+Math.random()*5),1.2+Math.random()*3.2,Math.sin(a)*(2+Math.random()*5),
            cc[0],cc[1],cc[2],.9+Math.random()*1.2,.55+Math.random()*.5,1.9);
        }
        if(impact>16)damage(impact*0.85,'HARD LANDING');
      }
      if(car.airTime>0.45){
        const bonus=Math.floor(car.airTime*140*combo);score+=bonus;
        toast('AIR +'+bonus,'#63e6ff');
      }
      car.airTime=0;
    }
  }
  if(!car.onGround)car.airTime+=dt;
  if(car.airTime>car.bestAir)car.bestAir=car.airTime;

  if(car.pos.y<p.water-0.2&&alive){
    damage(36*dt,'');car.vel.multiplyScalar(1-1.7*dt);
    if(Math.random()<dt*22){
      spawnParticle(car.pos.x+(Math.random()-.5)*2,p.water+.1,car.pos.z+(Math.random()-.5)*2,
        (Math.random()-.5)*2,2+Math.random()*3,(Math.random()-.5)*2,.55,.75,.85,.5,.5,2);
    }
  }

  car.driftAmount=clamp(Math.abs(vL)/7.5,0,1)*(car.onGround?1:0);
  if(alive&&car.driftAmount>0.22&&Math.abs(vF)>9){
    car.boost=Math.min(100,car.boost+dt*car.driftAmount*7.5);
    score+=dt*car.driftAmount*26*combo;
  }

  groundNormal(car.pos.x,car.pos.z,_nv);
  if(car.onGround)car.up.lerp(_nv,1-Math.exp(-9*dt));
  else car.up.lerp(_upv.set(0,1,0),1-Math.exp(-2.2*dt));
  car.up.normalize();
  rollS=damp(rollS,clamp(-vL*0.020,-0.20,0.20),7,dt);
  pitchS=damp(pitchS,clamp(accel*0.0072,-0.085,0.085),7,dt);
  buildCarQuat();
  carGroup.quaternion.slerp(_qT,1-Math.exp(-17*dt));
  carGroup.position.copy(car.pos);

  const spin=(vF/WHEEL_R)*dt;
  for(let i=0;i<4;i++){
    const w=wheels[i];
    w.spin.rotation.x-=spin*(hb&&!w.front?0.25:1);
    if(w.front)w.pivot.rotation.y=car.steer*(hb?1.15:1);
  }

  const kmh=Math.abs(vF)*3.6;
  if(alive&&Math.abs(vF)>car.topSpeed)car.topSpeed=Math.abs(vF);
  if(alive&&vF>0)distance+=vF*dt;
  if(alive)score+=Math.abs(vF)*dt*0.55*combo;

  if(alive&&car.onGround&&Math.abs(vF)>4){
    const inten=clamp(Math.abs(vF)/40+car.driftAmount*.8,0,1.4);
    const rate=inten*26*dt;let cnt=Math.floor(rate)+(Math.random()<(rate%1)?1:0);
    const cc=paramsAt(car.pos.z).cA;
    for(let i=0;i<cnt;i++){
      const w=wheels[2+((Math.random()<.5)?0:1)];
      const wp=new THREE.Vector3();w.pivot.getWorldPosition(wp);
      spawnParticle(wp.x,wp.y+.1,wp.z,
        -car.vel.x*.14+(Math.random()-.5)*2.2,.7+Math.random()*1.9,-car.vel.z*.14+(Math.random()-.5)*2.2,
        cc[0]*1.15,cc[1]*1.1,cc[2]*1.0,.55+Math.random()*.9,.65+Math.random()*.7,1.5);
    }
  }
  if(car.boosting&&alive){
    for(let i=0;i<3;i++){
      const bp=new THREE.Vector3((Math.random()-.5)*1.2,.55,-2.35).applyQuaternion(carGroup.quaternion).add(car.pos);
      spawnParticle(bp.x,bp.y,bp.z,-Math.sin(car.heading)*9+(Math.random()-.5)*2,1.4+Math.random()*1.6,
        -Math.cos(car.heading)*9+(Math.random()-.5)*2,1,.55+Math.random()*.3,.15,.42+Math.random()*.5,.28+Math.random()*.25,2.4);
    }
  }
  trailAccum+=Math.abs(vF)*dt;
  if(trailAccum>0.42){trailAccum=0;if(S.trail&&PQ().trail)pushTrail(runTime);}
  recordSample(dt);
}
function damage(amount,label){
  if(state!=='playing')return;
  car.hp-=amount;combo=1.0;
  dmgFxEl.style.opacity=Math.min(.95,.32+amount/60);
  setTimeout(()=>{dmgFxEl.style.opacity='0';},90);
  camShake=Math.min(1.4,camShake+amount*0.028);
  if(label)toast(label,'#ff4d4d');
  if(car.hp<=0){car.hp=0;endGame('Wrecked');}
}
function updateInteractions(dt){
  const alive=(state==='playing');
  nearby.length=0;
  const px=car.pos.x,pz=car.pos.z;
  for(let i=0;i<worldObjects.length;i++){
    const o=worldObjects[i];
    const dx=o.x-px,dz=o.z-pz;
    if(dx*dx+dz*dz<2025)nearby.push(o);
  }
  const p=paramsAt(pz);
  for(let i=0;i<nearby.length;i++){
    const o=nearby[i];
    if(o.t==='pad'){
      if(!o.mesh)o.mesh=getPad();
      o.mesh.position.set(o.x,o.y+.1+Math.sin(runTime*3+o.rot)*.03,o.z);
      o.mesh.rotation.z=-o.rot+Math.PI;
      o.mesh.material.opacity=o.cool>0?0.16:(0.72+0.2*Math.sin(runTime*5+o.rot));
      o.cool=Math.max(0,o.cool-dt);
    }
  }
  if(!alive)return;
  for(let i=0;i<nearby.length;i++){
    const o=nearby[i];
    const dx=o.x-px,dz=o.z-pz,d2=dx*dx+dz*dz;
    if(o.t==='coin'&&!o.got){
      if(d2<o.r*o.r&&Math.abs(o.y-car.pos.y)<3.4){
        o.got=true;coinsGot++;car.boost=Math.min(100,car.boost+11);score+=55*combo;
        SFX.coin();burst(o.x,o.y,o.z,10,1,.78,.20,5.5,.34,.5);
        for(const arr of cells.values()){const k=arr.indexOf(o);if(k>=0){arr.splice(k,1);break;}}
        rebuildFlat();
      }
    }else if(o.t==='pad'&&o.cool<=0){
      if(d2<o.r*o.r&&Math.abs(o.y-car.pos.y)<3.2){
        o.cool=3;car.boost=Math.min(100,car.boost+34);
        car.vel.x+=Math.sin(o.rot)*11;car.vel.z+=Math.cos(o.rot)*11;
        score+=40*combo;SFX.boostS();toast('BOOST PAD','#63e6ff');
        burst(o.x,o.y+.4,o.z,26,.45,.88,1,9,.5,.55);camShake=Math.min(1,camShake+.24);
      }
    }else if((o.t==='tree'||o.t==='rock')){
      const rr=o.r+1.05;
      if(d2<rr*rr){
        const d=Math.sqrt(d2)||0.001,nx=dx/d,nz=dz/d;
        const impact=Math.hypot(car.vel.x,car.vel.z);
        car.pos.x-=nx*(rr-d);car.pos.z-=nz*(rr-d);
        const dot=car.vel.x*nx+car.vel.z*nz;
        if(dot>0){car.vel.x-=nx*dot*1.55;car.vel.z-=nz*dot*1.55;}
        car.vel.multiplyScalar(o.t==='tree'?0.46:0.62);
        const dmg=clamp(impact*(o.t==='tree'?1.85:1.35),0,60);
        if(dmg>3){
          SFX.crash(impact);
          burst(car.pos.x+nx*1.2,car.pos.y+.8,car.pos.z+nz*1.2,20,
            o.t==='tree'?p.cT[0]:p.cR[0],o.t==='tree'?p.cT[1]:p.cR[1],o.t==='tree'?p.cT[2]:p.cR[2],7,.42,.6);
          damage(dmg,o.t==='tree'?'TREE IMPACT':'ROCK IMPACT');
        }
      }
    }else if(o.t==='bush'){
      const rr=o.r+0.9;
      if(d2<rr*rr){
        car.vel.multiplyScalar(1-clamp(7*dt,0,.4));
        if(Math.random()<0.3)burst(o.x,o.y+.5,o.z,1,p.cT[0],p.cT[1]*1.2,p.cT[2],3.5,.3,.6);
      }
    }
  }
  const ncp=nextCP();
  if(ncp){
    const dx=ncp.x-px,dz=ncp.z-pz;
    if(dx*dx+dz*dz<ncp.radius*ncp.radius&&Math.abs(ncp.y-car.pos.y)<13){
      ncp.taken=true;combo=Math.min(6,combo+0.35);
      const gained=Math.max(7.2,MAP.cpTime-cpCount*0.20);
      if(mode!=='infinite')timeLeft=Math.min(MAP.time*1.4,timeLeft+gained);
      const bonus=Math.floor((320+cpCount*45)*combo);
      score+=bonus;car.boost=Math.min(100,car.boost+18);
      SFX.cp();toast('CHECKPOINT +'+bonus+'  ×'+combo.toFixed(1),'#ffd23f');
      showCenter('CP '+cpCount,'+'+bonus+' BONUS',800);
      burst(ncp.x,ncp.y+3,ncp.z,42,1,.62,.16,13,.55,.9);
      camShake=Math.min(1,camShake+.18);
      maintainCheckpoints();
    }
  }
}

/* ══════════════════════════════════════════════════════════════
   19 · GAME STATE
   ══════════════════════════════════════════════════════════════ */
let state='boot',mode='stage';
let score=0,coinsGot=0,distance=0,timeLeft=40,combo=1,runTime=0,countdown=0,lastTickSec=-1;
let raceGhost=false,runStartMap=null;

function setMap(m,rebuildWorld){
  MAP=m;invalidateParams();
  if(rebuildWorld){
    releaseAllPads();releaseAllGates();cells.clear();rebuildFlat();
    resetCar();
    updateTerrain(car.pos.x,car.pos.z,true);
    refreshCells(car.pos.x,car.pos.z);rebuildProps();
  }
  applyMapVisuals();
}
function showScreen(id){
  ['mainMenu','mapSelect','settings','pause','over'].forEach(s=>$(s).classList.toggle('hidden',s!==id));
  $('topbar').classList.toggle('hide',id==='settings');
  if(id==='mainMenu'||id==='mapSelect'||id==='over'||id==='settings'){
    hudEl.style.display='none';
    if(id!=='over'){state='menu';enterPreview();}
  }
}
function enterPreview(){
  const m=(mode==='infinite')?INFINITE_MAP:MAPS[S.mapIdx]||MAPS[0];
  setMap(m,true);
  previewTarget.set(28,groundHeight(28,28),28);
  carGroup.position.copy(car.pos);
  camPos.set(previewTarget.x+40,previewTarget.y+16,previewTarget.z);
  camLook.copy(previewTarget);
  ghostCar.visible=false;$('ghostDelta').style.display='none';
}
function startGame(m,md,ghost){
  SFX.init();SFX.resume();
  mode=md;raceGhost=!!ghost;runStartMap=m;
  setMap(m,true);
  maintainCheckpoints();
  score=0;coinsGot=0;distance=0;combo=1;runTime=0;airTotal=0;
  timeLeft=(md==='infinite')?99999:m.time;
  camYaw=car.heading;camModeGhost=false;
  camPos.set(car.pos.x-Math.sin(car.heading)*10,car.pos.y+4,car.pos.z-Math.cos(car.heading)*10);
  if(raceGhost){ghPlay=loadGhost(m.id);}else{ghPlay=null;ghostCar.visible=false;$('ghostDelta').style.display='none';}
  ghRec=S.record?null:null;
  if(S.record)startRecord();else ghRec=null;
  countdown=3.6;lastTickSec=-1;state='countdown';
  showScreen(null);
  ['mainMenu','mapSelect','settings','pause','over'].forEach(s=>$(s).classList.add('hidden'));
  $('topbar').classList.add('hide');
  hudEl.style.display='block';
  $('ghostDelta').style.display='none';
  showCenter('3','Get Ready',1000);
}
function pauseGame(){if(state!=='playing')return;state='paused';$('pause').classList.remove('hidden');}
function resumeGame(){if(state!=='paused')return;state='playing';$('pause').classList.add('hidden');SFX.resume();}
let airTotal=0;
function endGame(reason){
  if(state==='over')return;
  state='over';
  const finalScore=Math.floor(score);
  const key=(mode==='infinite')?'infinite':(runStartMap?runStartMap.id:MAP.id);
  const prev=BESTS[key]||0;
  const isBest=finalScore>prev;
  if(isBest){BESTS[key]=finalScore;storeSet('rr.best',BESTS);}
  let saved=false;
  if(S.record&&mode!=='infinite'&&runStartMap&&(isBest||!loadGhost(runStartMap.id)))
    saved=saveGhost(runStartMap.id,finalScore,distance,cpCount);
  $('newBest').style.display=isBest?'block':'none';
  $('overReason').textContent=reason;
  $('rScore').textContent=finalScore.toLocaleString();
  $('rDist').textContent=Math.floor(distance).toLocaleString();
  $('rCp').textContent=cpCount;
  $('rCoin').textContent=coinsGot;
  $('rTop').textContent=Math.floor(convSpeed(car.topSpeed));
  $('rAir').textContent=car.bestAir.toFixed(1)+'s';
  $('rTime').textContent=Math.floor(runTime)+'s';
  let gd='—';
  if(ghPlay){
    const gt=ghPlay.n*GH_DT;
    gd=(runTime<gt)?'FINISHED':'DNF';
    const gdist=(ghPlay.dist||0);
    const dm=Math.floor(distance)-Math.floor(gdist);
    gd=(dm>=0?'+':'')+dm+'m';
  }else if(saved)gd='SET';
  $('rGhost').textContent=gd;
  $('bestOver').textContent=(BESTS[key]||0).toLocaleString();
  $('bestMenu').textContent=(BESTS[key]||0).toLocaleString();
  car.spinOut=(Math.random()<.5?-1:1)*2.6;
  setTimeout(()=>{if(state==='over'){showScreen('over');$('topbar').classList.remove('hide');}},1200);
  refreshMapCards();
}

/* ══════════════════════════════════════════════════════════════
   20 · MAP BROWSER UI
   ══════════════════════════════════════════════════════════════ */
const msGrid=$('msGrid');
let msFilter='all',msQuery='',msSel=0,thumbQueue=[],thumbDone=new Set();
function hasGhost(m){try{return !!localStorage.getItem(ghostKey(m.id));}catch(e){return false;}}
function buildMapCards(){
  msGrid.innerHTML='';thumbDone.clear();
  const frag=document.createDocumentFragment();
  const list=[INFINITE_MAP].concat(MAPS);
  list.forEach((m,i)=>{
    const id=m.infinite?'infinite':('m'+m.idx);
    const card=document.createElement('button');
    card.className='mapCard';card.dataset.id=id;card.dataset.idx=String(i-1);
    const cv=document.createElement('canvas');cv.className='thumb';cv.width=150;cv.height=86;
    card.appendChild(cv);
    const info=document.createElement('div');info.className='mcInfo';
    const diff=m.infinite?'∞':('●'.repeat(m.diff)+'<i>'+'●'.repeat(Math.max(0,5-m.diff))+'</i>');
    info.innerHTML='<div class="mcName">'+m.name+'</div><div class="mcMeta"><span class="pips">'+diff+
      '</span><span class="mcTheme">'+(m.infinite?'ENDLESS':m.themeLabel)+'</span></div>';
    card.appendChild(info);
    if(m.infinite){const b=document.createElement('div');b.className='mcBadge inf';b.textContent='∞ MODE';card.appendChild(b);}
    card.addEventListener('click',()=>selectMap(i-1,true));
    frag.appendChild(card);
  });
  msGrid.appendChild(frag);
  refreshMapCards();
  queueThumbs();
}
function refreshMapCards(){
  const cards=msGrid.children;
  for(let i=0;i<cards.length;i++){
    const c=cards[i],idx=parseInt(c.dataset.idx,10);
    const m=idx<0?INFINITE_MAP:MAPS[idx];
    const key=m.infinite?'infinite':m.id;
    let ok=true;
    if(msFilter==='easy'&&m.diff>2&&!m.infinite)ok=false;
    if(msFilter==='hard'&&m.diff<4&&!m.infinite)ok=false;
    if(msFilter==='ghost'&&!hasGhost(m))ok=false;
    if(msQuery&&m.name.toLowerCase().indexOf(msQuery)<0&&m.themeLabel.toLowerCase().indexOf(msQuery)<0)ok=false;
    c.style.display=ok?'':'none';
    c.classList.toggle('sel',idx===msSel);
    const oldB=c.querySelector('.mcBest'),oldG=c.querySelector('.mcBadge.gh');
    if(oldB)oldB.remove();if(oldG)oldG.remove();
    const b=BESTS[key];
    if(b){const d=document.createElement('div');d.className='mcBest';d.textContent=b.toLocaleString();c.appendChild(d);}
    if(hasGhost(m)&&!m.infinite){const g=document.createElement('div');g.className='mcBadge gh';g.textContent='GHOST';c.appendChild(g);}
  }
  let vis=0;for(let i=0;i<cards.length;i++)if(cards[i].style.display!=='none')vis++;
  $('msCount').textContent=vis+' STAGE'+(vis===1?'':'S');
}
function queueThumbs(){
  thumbQueue=[];
  const cards=msGrid.children;
  for(let i=0;i<cards.length;i++){
    const c=cards[i];if(c.style.display==='none')continue;
    const id=c.dataset.id;
    if(thumbDone.has(id))continue;
    thumbQueue.push({id,canvas:c.querySelector('canvas.thumb'),idx:parseInt(c.dataset.idx,10)});
  }
}
function processThumbQueue(budget){
  let n=0;
  while(thumbQueue.length&&n<budget){
    const item=thumbQueue.shift();
    renderThumb(item.idx<0?INFINITE_MAP:MAPS[item.idx],item.canvas);
    thumbDone.add(item.id);n++;
  }
}
function renderThumb(m,cv){
  const c=cv.getContext('2d'),W=cv.width,H=cv.height;
  const img=c.createImageData(W,H),d=img.data;
  const saveMAP=MAP;MAP=m;invalidateParams();
  const SUN=m.sunDir,len=Math.hypot(SUN[0],SUN[1],SUN[2])||1;
  const lx=SUN[0]/len,ly=SUN[1]/len,lz=SUN[2]/len;
  const span=560,x0=-span/2,z0=-span/2,st=span/W,sz=span/H;
  const pal=[m.cG,m.cD,m.cR,m.cK,m.cS,m.cA];
  const toSrgb=v=>v<=0.0031308?v*12.92:1.055*Math.pow(v,1/2.4)-0.055;
  for(let y=0;y<H;y++){
    for(let x=0;x<W;x++){
      const wx=x0+x*st,wz=z0+y*sz;
      const h=groundHeight(wx,wz);
      const hx=(groundHeight(wx+st,wz)-groundHeight(wx-st,wz))/(2*st);
      const hz=(groundHeight(wx,wz+sz)-groundHeight(wx,wz-sz))/(2*sz);
      let nx=-hx,ny=1,nz=-hz;const l=Math.hypot(nx,ny,nz);nx/=l;ny/=l;nz/=l;
      const slope=Math.sqrt(hx*hx+hz*hz);
      const hN=clamp((h-m.altLo)/Math.max(1,m.altHi-m.altLo),0,1);
      const biome=vnoise(wx*.0042+310,wz*.0042-190);
      let col=pal[0].slice();
      const dw=clamp(biome*1.55-.74,0,1)*.85;
      col=[col[0]+(pal[1][0]-col[0])*dw,col[1]+(pal[1][1]-col[1])*dw,col[2]+(pal[1][2]-col[2])*dw];
      const aw=smoothstep(.40,.76,hN);
      col=[col[0]+(pal[2][0]-col[0])*aw,col[1]+(pal[2][1]-col[1])*aw,col[2]+(pal[2][2]-col[2])*aw];
      const sn=smoothstep(m.snowLine,Math.min(.99,m.snowLine+.2),hN)*clamp(1.15-slope*1.5,0,1);
      if(sn>0)col=[col[0]+(pal[4][0]-col[0])*sn,col[1]+(pal[4][1]-col[1])*sn,col[2]+(pal[4][2]-col[2])*sn];
      const sw=smoothstep(.44,.88,slope)*.82;
      col=[col[0]+(pal[3][0]-col[0])*sw,col[1]+(pal[3][1]-col[1])*sw,col[2]+(pal[3][2]-col[2])*sw];
      const sh=smoothstep(m.water+3.4,m.water-.8,h)*.92;
      col=[col[0]+(pal[5][0]-col[0])*sh,col[1]+(pal[5][1]-col[1])*sh,col[2]+(pal[5][2]-col[2])*sh];
      let lam=nx*lx+ny*ly+nz*lz;lam=clamp(lam*0.85+0.30,0.18,1.5);
      let r,g,b;
      if(h<m.water){const dep=clamp((m.water-h)/26,0,1);
        r=m.cW[0]*(1-dep*.6);g=m.cW[1]*(1-dep*.5);b=m.cW[2]*(1-dep*.3);lam=clamp(lam*.7+.35,0,1.2);}
      else{r=col[0]*lam;g=col[1]*lam;b=col[2]*lam;}
      const fogW=smoothstep(H*0.05,H*0.62,y)*0.0;
      const o=(y*W+x)*4;
      d[o]=clamp(toSrgb(clamp(r,0,1)),0,1)*255;
      d[o+1]=clamp(toSrgb(clamp(g,0,1)),0,1)*255;
      d[o+2]=clamp(toSrgb(clamp(b,0,1)),0,1)*255;
      d[o+3]=255;
      if(fogW)d[o+3]=255;
    }
  }
  c.putImageData(img,0,0);
  c.fillStyle='rgba(0,0,0,0.28)';c.fillRect(0,H-16,W,16);
  MAP=saveMAP;invalidateParams();
}
function selectMap(idx,regen){
  msSel=idx;
  const m=idx<0?INFINITE_MAP:MAPS[idx];
  S.mapIdx=Math.max(0,idx);saveSettings();
  refreshMapCards();
  $('msName').textContent=m.name;
  $('msTheme').textContent=m.infinite?'ENDLESS PROCEDURAL':m.themeLabel;
  const tags=[];
  if(m.infinite)tags.push(['INFINITE STREAMING',1]);
  tags.push(['DIFFICULTY '+(m.infinite?'ADAPTIVE':m.diff+'/5'),m.diff>=4?1:0]);
  if(m.treeD>0.7)tags.push(['DENSE FOREST',0]);
  if(m.rockD>0.55)tags.push(['ROCKY',0]);
  if(m.water>-8)tags.push(['WET CROSSINGS',0]);
  if(m.ra>26)tags.push(['BIG ELEVATION',1]);
  if(hasGhost(m)&&!m.infinite)tags.push(['GHOST STORED',1]);
  $('msTags').innerHTML=tags.map(t=>'<span class="tag'+(t[1]?' hot':'')+'">'+t[0]+'</span>').join('');
  const relief=Math.round(m.a0+m.a1+m.ra*0.5);
  $('msStats').innerHTML=
    '<div><span>Relief</span><b>'+relief+'m</b></div>'+
    '<div><span>Time Limit</span><b>'+(m.infinite?'∞':m.time+'s')+'</b></div>'+
    '<div><span>Per CP</span><b>'+(m.infinite?'—':'+'+m.cpTime.toFixed(0)+'s')+'</b></div>'+
    '<div><span>Grip</span><b>'+(m.grip*100).toFixed(0)+'%</b></div>'+
    '<div><span>Vegetation</span><b>'+Math.round(m.treeD*100)+'%</b></div>'+
    '<div><span>Best Score</span><b>'+((BESTS[m.infinite?'infinite':m.id]||0)).toLocaleString()+'</b></div>';
  const g=m.infinite?null:loadGhost(m.id);
  $('msGhostLine').innerHTML=g?('⏱ GHOST AVAILABLE · '+g.dist.toLocaleString()+'m · '+g.score.toLocaleString()+' pts')
    :(m.infinite?'Endless biome rotation — no timer, chassis only.':'No ghost recorded on this stage yet.');
  $('msStart').textContent=m.infinite?'Enter Infinite':'Launch Stage';
  if(regen){
    setMap(m,true);
    previewTarget.set(28,groundHeight(28,28),28);
    camPos.set(previewTarget.x+42,previewTarget.y+15,previewTarget.z);
    camLook.copy(previewTarget);
  }
}
$('msSearch').addEventListener('input',e=>{msQuery=e.target.value.trim().toLowerCase();refreshMapCards();queueThumbs();});
Array.from(document.querySelectorAll('.chip[data-f]')).forEach(b=>{
  b.addEventListener('click',()=>{
    document.querySelectorAll('.chip[data-f]').forEach(x=>x.classList.remove('on'));
    b.classList.add('on');msFilter=b.dataset.f;refreshMapCards();queueThumbs();
  });
});
let msModeIdx=0;const MS_MODES=['stage','ghost'];
$('msMode').addEventListener('click',()=>{
  msModeIdx=(msModeIdx+1)%2;
  const mm=MS_MODES[msModeIdx];
  $('msMode').textContent='Mode: '+(mm==='stage'?'Stage':'Ghost Race');
  $('msMode').classList.toggle('on',mm==='ghost');
});
$('msStart').addEventListener('click',()=>{
  const m=msSel<0?INFINITE_MAP:MAPS[msSel];
  let md=(msSel<0)?'infinite':MS_MODES[msModeIdx];
  const ghost=(md==='ghost');
  if(ghost&&!hasGhost(m)){toast('No ghost stored yet — one will be recorded','#63e6ff');}
  startGame(m,md,ghost&&hasGhost(m));
});
$('msBack').addEventListener('click',()=>{showScreen('mainMenu');});
$('btnPlay').addEventListener('click',()=>{showScreen('mapSelect');selectMap(S.mapIdx,true);queueThumbs();});
$('btnInfinite').addEventListener('click',()=>{msSel=-1;selectMap(-1,true);showScreen('mapSelect');queueThumbs();});
$('btnSettings').addEventListener('click',()=>{prevScreen=curScreen();openSettings();});
$('btnSet').addEventListener('click',()=>{prevScreen=curScreen();openSettings();});
$('btnSound').addEventListener('click',toggleMute);
$('btnFull').addEventListener('click',toggleFullscreen);
$('btnResume').addEventListener('click',resumeGame);
$('btnRestart').addEventListener('click',()=>{const m=mode==='infinite'?INFINITE_MAP:(runStartMap||MAP);startGame(m,mode,raceGhost&&hasGhost(m));});
$('btnQuit').addEventListener('click',()=>{state='menu';$('pause').classList.add('hidden');enterPreview();showScreen('mainMenu');});
$('btnPauseSet').addEventListener('click',()=>{prevScreen='pause';openSettings();});
$('btnRetry').addEventListener('click',()=>{const m=mode==='infinite'?INFINITE_MAP:(runStartMap||MAP);startGame(m,mode,raceGhost&&hasGhost(m));});
$('btnStageSel').addEventListener('click',()=>{showScreen('mapSelect');selectMap(msSel,true);queueThumbs();});
$('btnMenu2').addEventListener('click',()=>{enterPreview();showScreen('mainMenu');});
$('setClose').addEventListener('click',closeSettings);

/* ══════════════════════════════════════════════════════════════
   21 · SETTINGS UI
   ══════════════════════════════════════════════════════════════ */
let prevScreen='mainMenu';
function curScreen(){
  for(const s of ['mainMenu','mapSelect','pause','over','settings'])
    if(!$(s).classList.contains('hidden'))return s;
  return 'mainMenu';
}
function openSettings(){$('settings').classList.remove('hidden');syncSettingsUI();}
function closeSettings(){
  $('settings').classList.add('hidden');
  if(prevScreen==='pause'){$('pause').classList.remove('hidden');}
  else if(state==='menu'){showScreen(prevScreen==='mapSelect'?'mapSelect':'mainMenu');}
}
function syncSettingsUI(){
  Array.from($('qualSeg').children).forEach(b=>b.classList.toggle('on',b.dataset.q===S.preset));
  $('swAuto').classList.toggle('on',!!S.auto);
  $('setRes').value=S.resScale;$('vRes').textContent=S.resScale+'%';
  $('setDraw').value=S.draw;$('vDraw').textContent=S.draw+'%';
  $('swShadow').classList.toggle('on',!!S.shadows);
  $('swBloom').classList.toggle('on',!!S.bloom);
  $('swTrail').classList.toggle('on',!!S.trail);
  $('setPart').value=S.part;$('vPart').textContent=S.part+'%';
  syncCamSeg();
  $('swAssist').classList.toggle('on',!!S.assist);
  $('setFov').value=S.fov;$('vFov').textContent=S.fov+'°';
  $('setShake').value=S.shake;$('vShake').textContent=S.shake+'%';
  Array.from($('unitSeg').children).forEach(b=>b.classList.toggle('on',b.dataset.u===S.units));
  $('swRecord').classList.toggle('on',!!S.record);
  $('setVol').value=S.vol;$('vVol').textContent=S.vol+'%';
  $('setEng').value=S.eng;$('vEng').textContent=S.eng+'%';
  $('setSfx').value=S.sfx;$('vSfx').textContent=S.sfx+'%';
}
function syncCamSeg(){Array.from($('camSeg').children).forEach(b=>b.classList.toggle('on',+b.dataset.c===S.cam));}
Array.from($('qualSeg').children).forEach(b=>b.addEventListener('click',()=>{
  S.preset=b.dataset.q;S.auto=false;saveSettings();syncSettingsUI();applyGraphics(true);
  toast('GRAPHICS · '+PQ().label,'#63e6ff');
}));
function bindSw(id,key,after){
  $(id).addEventListener('click',()=>{S[key]=!S[key];saveSettings();syncSettingsUI();if(after)after();});
}
bindSw('swAuto','auto');
bindSw('swShadow','shadows',()=>applyGraphics(false));
bindSw('swBloom','bloom',()=>{if(bloomPass)bloomPass.enabled=bloomOn&&S.bloom;});
bindSw('swTrail','trail');
bindSw('swAssist','assist');
bindSw('swRecord','record');
function bindRange(id,key,lab,after){
  $(id).addEventListener('input',e=>{
    S[key]=+e.target.value;$(lab).textContent=S[key]+(key==='fov'?'°':'%');
    saveSettings();if(after)after();
  });
}
bindRange('setRes','resScale','vRes',()=>onResize());
bindRange('setDraw','draw','vDraw',()=>applyGraphics(true));
bindRange('setPart','part','vPart',()=>{partScale=PQ().part*(S.part/100);});
bindRange('setFov','fov','vFov');
bindRange('setShake','shake','vShake');
bindRange('setVol','vol','vVol',()=>SFX.setVol());
bindRange('setEng','eng','vEng',()=>SFX.setVol());
bindRange('setSfx','sfx','vSfx',()=>SFX.setVol());
Array.from($('camSeg').children).forEach(b=>b.addEventListener('click',()=>{S.cam=+b.dataset.c;saveSettings();syncCamSeg();}));
Array.from($('unitSeg').children).forEach(b=>b.addEventListener('click',()=>{S.units=b.dataset.u;saveSettings();syncSettingsUI();}));
$('setReset').addEventListener('click',()=>{S=Object.assign({},DEF_SETTINGS);saveSettings();syncSettingsUI();applyGraphics(true);toast('SETTINGS RESET');});
$('btnBench').addEventListener('click',()=>{
  benching=true;benchT=0;benchFrames=0;$('btnBench').textContent='Benchmarking…';
});
let benching=false,benchT=0,benchFrames=0;

/* hardware info */
(function(){
  let gpu='unknown';
  try{
    const gl=renderer.getContext();
    const dbg=gl.getExtension('WEBGL_debug_renderer_info');
    if(dbg)gpu=gl.getParameter(dbg.UNMASKED_RENDERER_WEBGL)||'unknown';
  }catch(e){}
  $('sysInfo').innerHTML=
    'GPU: <b>'+String(gpu).slice(0,58)+'</b><br>'+
    'CPU threads: <b>'+(navigator.hardwareConcurrency||'?')+'</b><br>'+
    'Device memory: <b>'+(navigator.deviceMemory?navigator.deviceMemory+' GB':'n/a')+'</b><br>'+
    'Viewport: <b>'+innerWidth+'×'+innerHeight+' @'+(window.devicePixelRatio||1).toFixed(2)+'x</b><br>'+
    'Input: <b>'+(isTouch?'Touch':'Keyboard')+'</b> · Fullscreen: <b>'+
      (document.fullscreenEnabled||document.webkitFullscreenEnabled?'Yes':'No')+'</b><br>'+
    'Renderer: <b>three.js r160 · WebGL'+(renderer.capabilities.isWebGL2?'2':'1')+'</b>';
})();

/* ══════════════════════════════════════════════════════════════
   22 · ADAPTIVE PERFORMANCE
   ══════════════════════════════════════════════════════════════ */
let fpsAcc=0,fpsFrames=0,fpsWin=0,fpsNow=60,lowStreak=0,highStreak=0;
function perfTick(dt){
  fpsAcc+=dt;fpsFrames++;
  if(fpsAcc>=0.5){
    fpsNow=fpsFrames/fpsAcc;fpsAcc=0;fpsFrames=0;
    $('fpsLive').textContent=Math.round(fpsNow)+' fps';
    $('fpsLive').style.color=fpsNow>=50?'#5ce07a':(fpsNow>=32?'#ffd23f':'#ff4d4d');
    if(benching){
      benchT+=0.5;benchFrames+=Math.round(fpsNow*0.5);
      if(benchT>=3){
        benching=false;
        const avg=benchFrames/3;
        let rec='med';
        if(avg>=105)rec='best';else if(avg>=78)rec='high';else if(avg>=55)rec='med';
        else if(avg>=36)rec='low';else rec='bare';
        S.preset=rec;S.auto=true;saveSettings();syncSettingsUI();applyGraphics(true);
        $('btnBench').textContent='Run 3s Benchmark';
        toast('RECOMMENDED · '+PQ().label+' ('+Math.round(avg)+'fps)','#63e6ff');
      }
      return;
    }
    if(!S.auto)return;
    if(state==='playing'||state==='countdown'){
      if(fpsNow<44){lowStreak++;highStreak=0;}
      else if(fpsNow>57){highStreak++;lowStreak=0;}
      else{lowStreak=0;highStreak=0;}
      if(lowStreak>=3){
        lowStreak=0;
        if(dynScale>0.66){dynScale=Math.max(0.62,dynScale-0.12);onResize();}
        else{
          const order=['bare','low','med','high','best'];
          const i=order.indexOf(S.preset);
          if(i>0){S.preset=order[i-1];saveSettings();syncSettingsUI();applyGraphics(true);
            toast('AUTO · '+PQ().label,'#ffd23f');}
        }
      }else if(highStreak>=10){
        highStreak=0;
        const q=PQ();
        const base=Math.min(window.devicePixelRatio||1,q.dpr);
        if(dynScale<1-1e-3){dynScale=Math.min(1,dynScale+0.08);onResize();}
      }
    }
  }
}

/* ══════════════════════════════════════════════════════════════
   23 · MAIN LOOP
   ══════════════════════════════════════════════════════════════ */
const PHYS_DT=1/120;
let acc=0,lastT=0;
function frame(now){
  requestAnimationFrame(frame);
  let dt=(now-lastT)/1000;lastT=now;
  if(!isFinite(dt)||dt<=0)dt=0.016;
  if(dt>0.1)dt=0.1;
  perfTick(dt);

  if(state==='countdown'){
    countdown-=dt;
    const s=Math.ceil(countdown);
    if(s!==lastTickSec){
      lastTickSec=s;
      if(s>0&&s<=3){showCenter(String(s),'Get Ready',950);SFX.tick();}
      else if(s<=0){showCenter('GO!','',750);SFX.blip(1320,.35,'square',.18);state='playing';}
    }
  }
  if(state==='playing'){
    runTime+=dt;
    if(mode!=='infinite'){
      timeLeft-=dt;
      if(timeLeft<=0){timeLeft=0;endGame('Time Up');}
      if(timeLeft<6&&Math.floor(timeLeft)!==Math.floor(timeLeft+dt)&&Math.floor(timeLeft)>0)SFX.tick();
    }
    if(Math.abs(car.vForward)<4&&combo>1)combo=Math.max(1,combo-dt*0.16);
  }

  const running=(state==='playing'||state==='countdown'||state==='over');
  if(running){
    acc+=dt;let steps=0;
    while(acc>=PHYS_DT&&steps<8){physics(PHYS_DT);acc-=PHYS_DT;steps++;}
    if(acc>PHYS_DT*8)acc=0;
  }else{
    car.pos.y=groundHeight(car.pos.x,car.pos.z)+CAR_RIDE;
    groundNormal(car.pos.x,car.pos.z,_nv);
    car.up.lerp(_nv,1-Math.exp(-6*dt));car.up.normalize();
    buildCarQuat();carGroup.quaternion.slerp(_qT,1-Math.exp(-10*dt));
    carGroup.position.copy(car.pos);
  }

  updateTerrain(car.pos.x,car.pos.z,false);
  refreshCells(car.pos.x,car.pos.z);
  rebuildProps();
  updateCoins(dt,runTime);
  updateInteractions(dt);
  updateParticles(dt);
  rebuildTrail(runTime);

  if(running||state==='paused'){
    updateCamera(dt);
    updateGhost(dt);
    if(state==='playing'||state==='countdown')maintainCheckpoints();
  }else{
    updatePreviewCam(dt);
    ghostCar.visible=false;
  }

  for(let i=0;i<checkpoints.length;i++){
    const cp=checkpoints[i],isN=cp===nextCP();
    if(cp.mesh.userData.ring){
      cp.mesh.userData.ring.rotation.z+=dt*(isN?2.1:0.5);
      cp.mesh.userData.ring.scale.setScalar(isN?(1+Math.sin(runTime*5)*0.08):0.75);
      cp.mesh.userData.ground.material.opacity=isN?(0.20+0.16*Math.sin(runTime*4)):0.05;
    }
  }

  sun.position.set(car.pos.x+sunDirV.x*140,car.pos.y+sunDirV.y*140,car.pos.z+sunDirV.z*140);
  sun.target.position.set(car.pos.x,car.pos.y,car.pos.z);
  sun.target.updateMatrixWorld();
  if(beamL)beamL.intensity=(state==='playing')?2.2:0;

  const rpm=clamp(Math.abs(car.vForward)/MAX_SPEED,0,1)*.72+(keys.up?.22:.06)+Math.random()*.02;
  SFX.engine(rpm,keys.up?1:.25,Math.abs(car.vForward),running&&state!=='over');

  hudAcc+=dt;
  if(hudAcc>0.05&&hudEl.style.display!=='none'){hudAcc=0;tickHUD();}
  if(!$('mapSelect').classList.contains('hidden'))processThumbQueue(isSmallScreen()?1:2);

  if(composer)composer.render(dt);else renderer.render(scene,camera);
}

/* ══════════════════════════════════════════════════════════════
   24 · BOOT
   ══════════════════════════════════════════════════════════════ */
function setBoot(pct,txt){$('bootFill').style.width=pct+'%';if(txt)$('bootTxt').textContent=txt;}
function boot(){
  try{
    setBoot(20,'Compiling shaders…');
    buildTerrain();
    setBoot(45,'Seeding 56 stages…');
    MAP=MAPS[S.mapIdx]||MAPS[0];
    msSel=S.mapIdx;
    resetCar();
    setBoot(65,'Streaming terrain…');
    updateTerrain(car.pos.x,car.pos.z,true);
    refreshCells(car.pos.x,car.pos.z);
    rebuildProps();
    setBoot(80,'Applying graphics…');
    applyGraphics(false);
    buildMapCards();
    selectMap(S.mapIdx,false);
    setBoot(92,'Calibrating…');
    applyMapVisuals();
    onResize();
    syncSettingsUI();
    $('bestMenu').textContent=(BESTS[MAP.id]||0).toLocaleString();
    renderer.compile(scene,camera);
    previewTarget.set(28,groundHeight(28,28),28);
    camPos.set(previewTarget.x+42,previewTarget.y+15,previewTarget.z);
    camLook.copy(previewTarget);
    camera.position.copy(camPos);camera.lookAt(camLook);
    sky.position.copy(camera.position);
    setBoot(100,'Ready');
    state='menu';
    showScreen('mainMenu');
    $('boot').classList.add('gone');
    setTimeout(()=>{const b=$('boot');if(b)b.style.display='none';},600);
    lastT=performance.now();
    requestAnimationFrame(frame);
    setTimeout(tryPost,300);
  }catch(err){
    console.error(err);
    $('bootTxt').textContent='Startup failed';
    $('errBox').style.display='block';
    $('errBox').textContent=(err&&err.message)?err.message:String(err);
    document.querySelector('.spin').style.display='none';
  }
}
addEventListener('error',ev=>{
  const b=$('errBox');
  if(b&&!b.textContent){b.style.display='block';b.textContent='Error: '+(ev.message||'unknown');}
});
addEventListener('pointerdown',()=>{SFX.init();SFX.resume();},{once:true});
boot();
</script>
</body>
</html>
