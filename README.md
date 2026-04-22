<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Stellar Drift 3D</title>
<style>
*{box-sizing:border-box}
body{margin:0;overflow:hidden;background:#000;font-family:'Segoe UI',Arial,sans-serif;color:#fff}
#renderer-container{position:fixed;top:0;left:0;width:100%;height:100%;z-index:0}
#renderer-container canvas{display:block;width:100%!important;height:100%!important}

/* ── SHARED SCREEN SHELL ── */
#homeMain,#boostsScreen,#questsScreen{
  position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);
  z-index:100;
  background:rgba(2,4,20,.95);border:1px solid #00ffcc33;
  border-radius:16px;
  width:min(520px,96vw);max-height:93vh;overflow-y:auto;
  scrollbar-width:thin;scrollbar-color:#00ffcc22 transparent;
}
#homeMain::-webkit-scrollbar,#boostsScreen::-webkit-scrollbar,#questsScreen::-webkit-scrollbar{width:4px}
#homeMain::-webkit-scrollbar-thumb,#boostsScreen::-webkit-scrollbar-thumb,#questsScreen::-webkit-scrollbar-thumb{background:#00ffcc22;border-radius:3px}

/* ── HOME MAIN ── */
#homeMain{padding:14px 16px 16px;text-align:center}
#hmTop{display:flex;align-items:center;justify-content:space-between;margin-bottom:8px;gap:8px;flex-wrap:wrap}
#hmPilot{flex:0 0 auto}
#playerNameInput{background:#111;border:1px solid #00ffcc44;color:#fff;border-radius:6px;padding:4px 10px;font-size:12px;width:110px;text-align:center}
#hmRankBlock{flex:1;text-align:center}
#hmRankBlock .rankLabel{font-size:18px;margin-bottom:2px}
#hmGems{flex:0 0 auto;font-size:14px}

/* selected ship card */
#hmShipCard{
  display:flex;align-items:center;justify-content:center;gap:14px;
  background:rgba(255,255,255,.04);border:1px solid #00ffcc22;
  border-radius:12px;padding:12px 16px;margin:10px 0;
  cursor:pointer;transition:border-color .2s,background .2s;
}
#hmShipCard:hover{border-color:#00ffcc55;background:rgba(0,255,204,.06)}
#hmShipSvg{flex:0 0 70px}
#hmShipInfo{text-align:left;flex:1}
#hmShipName{font-size:15px;font-weight:bold;color:#fff;margin-bottom:3px}
#hmShipPower{font-size:11px;color:#00ffcc;opacity:.85}

/* ── BRAWL STARS PLAY BUTTON ── */
#startBtn{
  display:block;width:80%;margin:0 auto 12px;
  padding:15px 0;font-size:24px;font-weight:900;
  border:3px solid #b8860b;
  background:linear-gradient(180deg,#ffe135 0%,#ffc200 45%,#e89900 100%);
  color:#2a1a00;border-radius:14px;cursor:pointer;
  text-shadow:0 1px 0 rgba(255,255,255,0.4);
  box-shadow:0 4px 0 #8a6000,0 6px 18px rgba(255,180,0,0.45),inset 0 1px 0 rgba(255,255,255,0.35);
  transition:all .1s;letter-spacing:3px;
  position:relative;top:0;
}
#startBtn:hover{
  background:linear-gradient(180deg,#ffec55 0%,#ffd020 45%,#f0a800 100%);
  box-shadow:0 4px 0 #8a6000,0 8px 22px rgba(255,180,0,0.55),inset 0 1px 0 rgba(255,255,255,0.4);
}
#startBtn:active{top:3px;box-shadow:0 1px 0 #8a6000,0 3px 10px rgba(255,180,0,0.35);}

/* ── NAV BUTTONS — each a different color ── */
#hmNav{display:flex;gap:8px;justify-content:center;margin-bottom:10px}
.hmNavBtn{
  flex:1;max-width:130px;padding:9px 0;font-size:13px;font-weight:bold;
  border-radius:9px;cursor:pointer;border:none;
  transition:filter .15s,box-shadow .15s;
}
#navHangar{background:linear-gradient(135deg,#0055bb,#0088ff);color:#fff;box-shadow:0 2px 8px rgba(0,136,255,.35);}
#navHangar:hover{filter:brightness(1.15);box-shadow:0 3px 12px rgba(0,136,255,.55);}
#navBoosts{background:linear-gradient(135deg,#cc4400,#ff6600);color:#fff;box-shadow:0 2px 8px rgba(255,102,0,.35);}
#navBoosts:hover{filter:brightness(1.15);box-shadow:0 3px 12px rgba(255,102,0,.55);}
#navQuests{background:linear-gradient(135deg,#5500aa,#8844dd);color:#fff;box-shadow:0 2px 8px rgba(136,68,221,.35);}
#navQuests:hover{filter:brightness(1.15);box-shadow:0 3px 12px rgba(136,68,221,.55);}
#navShop{background:linear-gradient(135deg,#006633,#00bb55);color:#fff;box-shadow:0 2px 8px rgba(0,180,80,.35);}
#navShop:hover{filter:brightness(1.15);box-shadow:0 3px 12px rgba(0,180,80,.55);}
#navPass{
  background:linear-gradient(135deg,#6600cc,#ff00aa,#ffcc00);
  background-size:200% 200%;
  animation:passNavShift 3s ease infinite;
  color:#fff;box-shadow:0 2px 8px rgba(180,0,200,.45);
  text-shadow:0 1px 3px rgba(0,0,0,.5);
}
#navPass:hover{filter:brightness(1.15);}
@keyframes passNavShift{0%{background-position:0% 50%}50%{background-position:100% 50%}100%{background-position:0% 50%}}

/* ── CHROMATIC RARITY ── */
@keyframes chromaticShift{
  0%  {color:#ff4488;text-shadow:0 0 8px #ff4488;}
  16% {color:#ff8800;text-shadow:0 0 8px #ff8800;}
  33% {color:#ffee00;text-shadow:0 0 8px #ffee00;}
  50% {color:#00ffaa;text-shadow:0 0 8px #00ffaa;}
  66% {color:#00ccff;text-shadow:0 0 8px #00ccff;}
  83% {color:#cc44ff;text-shadow:0 0 8px #cc44ff;}
  100%{color:#ff4488;text-shadow:0 0 8px #ff4488;}
}
@keyframes chromaticBorder{
  0%  {border-color:#ff4488;box-shadow:0 0 10px #ff448866;}
  16% {border-color:#ff8800;box-shadow:0 0 10px #ff880066;}
  33% {border-color:#ffee00;box-shadow:0 0 10px #ffee0066;}
  50% {border-color:#00ffaa;box-shadow:0 0 10px #00ffaa66;}
  66% {border-color:#00ccff;box-shadow:0 0 10px #00ccff66;}
  83% {border-color:#cc44ff;box-shadow:0 0 10px #cc44ff66;}
  100%{border-color:#ff4488;box-shadow:0 0 10px #ff448866;}
}
@keyframes chromaticBg{
  0%  {background:linear-gradient(135deg,#1a0010,#0a0018)}
  25% {background:linear-gradient(135deg,#001a0a,#001815)}
  50% {background:linear-gradient(135deg,#0a1a00,#001505)}
  75% {background:linear-gradient(135deg,#00101a,#000c1a)}
  100%{background:linear-gradient(135deg,#1a0010,#0a0018)}
}
.rarity-chromatic{
  animation:chromaticShift 2.5s linear infinite;
  background:transparent!important;border:1px solid transparent;
  animation:chromaticShift 2.5s linear infinite,chromaticBorder 2.5s linear infinite;
  padding:1px 7px;border-radius:4px;font-size:10px;font-weight:bold;letter-spacing:.8px;display:inline-block;margin-bottom:5px;
}
.skinCard.chromatic-card{animation:chromaticBg 4s ease infinite,chromaticBorder 2.5s linear infinite;border-width:2px!important;}

/* ── STELLAR PASS SCREEN ── */
#passScreen{
  position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);
  z-index:100;
  background:rgba(4,2,20,.97);
  border-radius:16px;
  width:min(520px,96vw);max-height:93vh;overflow-y:auto;
  scrollbar-width:thin;scrollbar-color:#cc44ff22 transparent;
  padding:0;
  animation:chromaticBorder 3s linear infinite;border:2px solid #cc44ff;
}
#passScreen::-webkit-scrollbar{width:4px}
#passScreen::-webkit-scrollbar-thumb{background:#cc44ff33;border-radius:3px}
#passHeader{
  display:flex;align-items:center;justify-content:space-between;
  padding:10px 14px;border-bottom:1px solid #cc44ff22;
  position:sticky;top:0;z-index:2;
  background:rgba(4,2,20,.98);border-radius:16px 16px 0 0;
}
#passTitle{font-size:16px;font-weight:bold;animation:chromaticShift 2.5s linear infinite;}
#passGemCount{font-size:12px;color:#888}
#passBody{padding:12px 14px 18px}
#passBanner{
  border-radius:12px;padding:14px 16px;margin-bottom:12px;text-align:center;
  animation:chromaticBg 4s ease infinite,chromaticBorder 2.5s linear infinite;
  border:2px solid transparent;
}
#passBannerTitle{font-size:22px;font-weight:900;letter-spacing:2px;animation:chromaticShift 2.5s linear infinite;}
#passBannerSub{font-size:11px;color:#aaa;margin-top:4px}
#passXpBar{margin:10px 0 14px}
#passXpLabel{font-size:11px;color:#888;margin-bottom:4px;display:flex;justify-content:space-between}
#passXpTrack{height:12px;background:#0d0820;border-radius:6px;overflow:hidden;border:1px solid #330055}
#passXpFill{height:100%;border-radius:6px;transition:width .4s;background:linear-gradient(90deg,#cc44ff,#ff44bb,#ffcc00)}
#passTiersGrid{display:flex;flex-direction:column;gap:8px}
.passTier{
  display:flex;align-items:center;gap:10px;
  background:#07040f;border:1px solid #1a0035;border-radius:10px;
  padding:9px 12px;transition:border-color .2s;
}
.passTier.tier-unlocked{border-color:#330055;}
.passTier.tier-claimed{border-color:#006622;background:#040e06;}
.passTier.tier-available{animation:chromaticBorder 2.5s linear infinite;background:#0d0515;}
.tierNum{
  width:32px;height:32px;border-radius:50%;display:flex;align-items:center;justify-content:center;
  font-size:13px;font-weight:bold;flex-shrink:0;
  border:2px solid #330055;background:#0a0518;color:#776;
}
.passTier.tier-claimed .tierNum{background:#003311;border-color:#00cc44;color:#00ff66;}
.passTier.tier-available .tierNum{animation:chromaticBorder 2.5s linear infinite;color:#fff;background:#1a0030;}
.tierReward{flex:1}
.tierRewardName{font-size:13px;font-weight:bold;color:#ccc}
.passTier.tier-claimed .tierRewardName{color:#44aa66}
.tierRewardDesc{font-size:10px;color:#556;margin-top:1px}
.tierXpReq{font-size:10px;color:#445;text-align:right;flex-shrink:0}
.passTier.tier-claimed .tierXpReq{color:#225}
.tierClaimBtn{
  padding:5px 14px;font-size:12px;font-weight:bold;border-radius:7px;cursor:pointer;border:none;flex-shrink:0;
  animation:chromaticBorder 2.5s linear infinite;
  background:linear-gradient(135deg,#220033,#110022);
  color:#fff;transition:filter .15s;
}
.tierClaimBtn:hover{filter:brightness(1.2);}
.tierClaimedBadge{font-size:11px;color:#00cc44;font-weight:bold;flex-shrink:0;}
.tierLockedBadge{font-size:18px;flex-shrink:0;opacity:.4}
#passFooter{margin-top:14px;padding:10px 12px;background:#070410;border-radius:10px;border:1px solid #1a0035;font-size:11px;color:#556;text-align:center;line-height:1.7}

/* ── SHOP SCREEN ── */
#shopScreen{
  position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);
  z-index:100;
  background:rgba(2,4,20,.97);border:1px solid #00bb5544;
  border-radius:16px;
  width:min(520px,96vw);max-height:93vh;overflow-y:auto;
  scrollbar-width:thin;scrollbar-color:#00bb5522 transparent;
  padding:0;
}
#shopScreen::-webkit-scrollbar{width:4px}
#shopScreen::-webkit-scrollbar-thumb{background:#00bb5522;border-radius:3px}
#shopHeader{
  display:flex;align-items:center;justify-content:space-between;
  padding:10px 14px;border-bottom:1px solid #00bb5522;position:sticky;top:0;z-index:2;
  background:rgba(2,4,20,.98);border-radius:16px 16px 0 0;
}
#shopTitle{font-size:15px;font-weight:bold;color:#00ff88}
#shopTimerBox{
  display:flex;align-items:center;gap:6px;
  background:#0a1a0e;border:1px solid #00bb5533;border-radius:7px;padding:4px 10px;
  font-size:12px;color:#00ff88;font-weight:bold;
}
#shopTimerBox .timerIcon{font-size:14px}
#shopTimerBox .timerVal{font-family:monospace;font-size:13px;color:#66ffaa;min-width:44px;text-align:right}
#shopBody{padding:12px 14px 16px}
#shopSubtitle{font-size:11px;color:#556;margin:0 0 10px;text-align:center}
#shopCards{display:flex;flex-direction:column;gap:10px}
.skinCard{
  display:flex;align-items:center;gap:12px;
  background:#070f18;border:1px solid #223;border-radius:12px;padding:11px 14px;
  transition:border-color .2s,background .2s;position:relative;
}
.skinCard:hover{border-color:#00bb5544;background:#0a1510}
.skinCard.owned-equipped{border-color:#00ff88;background:#071409;}
.skinCard.owned-unequipped{border-color:#44aa66;background:#060e09;}
.skinCard.not-owned{border-color:#223;}
.skinColorSwatch{
  width:52px;height:52px;border-radius:10px;flex-shrink:0;
  display:flex;align-items:center;justify-content:center;
  border:2px solid rgba(255,255,255,.1);
  box-shadow:inset 0 0 16px rgba(0,0,0,.4);
  transition:box-shadow .2s;
}
.skinCard:hover .skinColorSwatch{box-shadow:inset 0 0 10px rgba(0,0,0,.3),0 0 14px var(--sw-glow,rgba(255,255,255,.2));}
.skinInfo{flex:1;min-width:0}
.skinShipName{font-size:10px;color:#556;font-weight:bold;letter-spacing:.5px;text-transform:uppercase;margin-bottom:2px}
.skinName{font-size:15px;font-weight:bold;color:#fff;margin-bottom:3px}
.skinRarity{font-size:10px;font-weight:bold;letter-spacing:.8px;border-radius:4px;padding:1px 7px;display:inline-block;margin-bottom:5px}
.rarity-rare{background:rgba(68,136,255,.15);color:#66aaff;border:1px solid #4488ff44}
.rarity-epic{background:rgba(180,68,255,.15);color:#cc66ff;border:1px solid #bb44ff44}
.rarity-legendary{background:rgba(255,180,0,.12);color:#ffcc44;border:1px solid #ffcc0044}
.rarity-mythic{background:rgba(255,60,60,.12);color:#ff8888;border:1px solid #ff444444}
.skinActions{display:flex;flex-direction:column;align-items:flex-end;gap:5px;flex-shrink:0}
.skinCost{font-size:12px;color:#ffd700;font-weight:bold}
.skinBuyBtn{
  padding:6px 14px;font-size:12px;font-weight:bold;border-radius:7px;cursor:pointer;
  border:2px solid #b8860b;background:linear-gradient(135deg,#2a1800,#3d2600);color:#ffd700;
  box-shadow:0 2px 6px rgba(255,180,0,.2);transition:filter .15s;
}
.skinBuyBtn:hover{filter:brightness(1.15);}
.skinBuyBtn:disabled{border-color:#333;background:#111;color:#444;cursor:not-allowed;box-shadow:none;filter:none;}
.skinEquipBtn{
  padding:6px 14px;font-size:12px;font-weight:bold;border-radius:7px;cursor:pointer;
  border:2px solid #00cc44;background:linear-gradient(135deg,#003311,#004d1a);color:#00ff66;
  transition:filter .15s;
}
.skinEquipBtn:hover{filter:brightness(1.15);}
.skinEquipBtn.equipped{border-color:#ffd700;background:linear-gradient(135deg,#2a1800,#3d2600);color:#ffd700;cursor:default;filter:none;}
.skinUnequipBtn{
  padding:4px 10px;font-size:11px;font-weight:bold;border-radius:6px;cursor:pointer;
  border:1px solid #ff4444aa;background:#2a0808;color:#ff6666;transition:filter .15s;
}
.skinUnequipBtn:hover{filter:brightness(1.2);}
.skinOwnedBadge{position:absolute;top:8px;right:8px;font-size:9px;font-weight:bold;background:#00cc44;color:#fff;border-radius:4px;padding:1px 5px;}

/* ── HANGAR SCREEN — FIX: proper flex+overflow for scrolling ── */
#hangarScreen{
  position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);
  z-index:100;
  background:rgba(2,4,20,.95);border:1px solid #00ffcc33;
  border-radius:16px;
  width:min(520px,96vw);
  padding:0;overflow:hidden;display:flex;flex-direction:column;
  height:min(600px,92vh);max-height:92vh;
}
#hangarHeader{
  display:flex;align-items:center;justify-content:space-between;
  padding:10px 14px;border-bottom:1px solid #00ffcc22;flex-shrink:0;
}
#hangarTitle{font-size:15px;font-weight:bold;color:#00ffcc}
/* Back button — red tint */
#hangarBack{
  padding:5px 12px;font-size:12px;border-radius:7px;cursor:pointer;
  border:1px solid #ff4444aa;background:#2a0808;color:#ff6666;font-weight:bold;
  transition:background .15s;
}
#hangarBack:hover{background:#3a1010;}
/* Booster button — gold */
#boosterBtn{
  padding:5px 12px;font-size:12px;border-radius:7px;cursor:pointer;
  border:2px solid #b8860b;background:linear-gradient(135deg,#2a1800,#3d2600);
  color:#ffd700;font-weight:bold;box-shadow:0 2px 8px rgba(255,180,0,.25);
  transition:background .15s,box-shadow .15s;
}
#boosterBtn:hover{background:linear-gradient(135deg,#3d2600,#553500);box-shadow:0 3px 12px rgba(255,180,0,.4);}
#boosterBtn:disabled{border-color:#444;color:#555!important;background:#1a1a1a;box-shadow:none;cursor:not-allowed;opacity:.5;}
#hangarBody{display:flex;flex:1;overflow:hidden;min-height:0;}

/* ship list column — FIX: min-height:0 enables overflow scroll inside flex */
#hangarList{
  width:110px;flex-shrink:0;overflow-y:auto;min-height:0;
  border-right:1px solid #00ffcc11;padding:6px 4px;
  scrollbar-width:thin;scrollbar-color:#00ffcc33 transparent;
}
#hangarList::-webkit-scrollbar{width:4px}
#hangarList::-webkit-scrollbar-thumb{background:#00ffcc33;border-radius:3px}
.hListItem{
  display:flex;flex-direction:column;align-items:center;
  padding:6px 4px;border-radius:8px;cursor:pointer;margin-bottom:4px;
  border:1px solid transparent;transition:background .15s,border-color .15s;
  position:relative;
}
.hListItem:hover{background:rgba(0,255,204,.07);border-color:#00ffcc22}
.hListItem.selected{background:rgba(0,255,204,.12);border-color:#00ffcc66}
.hListItem.g2item{border-color:rgba(221,68,255,.1)}
.hListItem.g2item.selected{background:rgba(221,68,255,.12);border-color:#dd44ff88}
.hListItem.locked{opacity:.35;cursor:default}
.hListItem.pass-item{border-color:rgba(180,50,255,.15);}
.hListItem.pass-item.selected{animation:chromaticBorder 2.5s linear infinite;background:rgba(180,50,255,.1);}
.hListItemName{font-size:8px;color:#aaa;margin-top:3px;text-align:center;line-height:1.2}
.hListBoostBadge{position:absolute;top:2px;right:3px;background:#0ff;color:#111;font-size:7px;font-weight:bold;border-radius:3px;padding:1px 3px}
.hListG2Badge{position:absolute;top:2px;left:3px;background:#dd44ff;color:#fff;font-size:7px;font-weight:bold;border-radius:3px;padding:1px 3px}
.hListSep{width:100%;text-align:center;font-size:8px;color:#440066;padding:6px 0 3px;letter-spacing:.5px;border-top:1px solid #330044;margin-top:4px}

/* detail column — FIX: min-height:0 + own scroll */
#hangarDetail{
  flex:1;min-width:0;min-height:0;
  display:flex;flex-direction:column;align-items:center;
  padding:10px 12px;overflow-y:auto;
  scrollbar-width:thin;scrollbar-color:#00ffcc22 transparent;
}
#hangarDetail::-webkit-scrollbar{width:4px}
#hangarDetail::-webkit-scrollbar-thumb{background:#00ffcc22;border-radius:3px}
#shipPreviewCanvas{
  width:180px;height:180px;border-radius:12px;
  background:radial-gradient(ellipse at center,#0a0f25 0%,#020408 100%);
  border:1px solid #00ffcc22;flex-shrink:0;display:block;
}
#hdName{font-size:17px;font-weight:bold;color:#fff;margin:8px 0 2px;text-align:center}
#hdGalaxy{font-size:10px;font-weight:bold;letter-spacing:1px;margin-bottom:6px}
#hdPower{font-size:13px;font-weight:bold;color:#00ffcc;margin-bottom:4px;text-align:center}
#hdPowerDesc{font-size:11px;color:#8ab;line-height:1.5;text-align:center;margin-bottom:6px;max-width:240px}
#hdUnlock{font-size:11px;margin-bottom:4px}
#hdBoostDesc{font-size:10px;color:#0ff;background:rgba(0,255,255,.07);border-radius:6px;padding:4px 8px;margin-bottom:8px;display:none;text-align:center;max-width:240px}
/* Select button — green */
#hdSelectBtn{
  padding:9px 28px;font-size:14px;font-weight:bold;
  border:2px solid #00cc44;background:linear-gradient(135deg,#003311,#004d1a);color:#00ff66;
  border-radius:9px;cursor:pointer;transition:background .15s,box-shadow .15s;
  box-shadow:0 2px 10px rgba(0,204,68,.25);flex-shrink:0;margin-bottom:8px;
}
#hdSelectBtn:hover{background:linear-gradient(135deg,#004d1a,#006622);box-shadow:0 3px 14px rgba(0,204,68,.4);}
#hdSelectBtn.isSelected{border-color:#ffd700;color:#ffd700;background:linear-gradient(135deg,#2a1800,#3d2600);box-shadow:0 2px 10px rgba(255,180,0,.25);cursor:default;}
#hdSelectBtn:disabled:not(.isSelected){border-color:#444;color:#555;background:#1a1a1a;box-shadow:none;cursor:not-allowed;}

/* ── BOOSTS/QUESTS SCREENS ── */
#boostsScreen,#questsScreen{padding:12px 14px}
#boostsScreenHeader,#questsScreenHeader{
  display:flex;align-items:center;justify-content:space-between;
  margin-bottom:8px;padding-bottom:8px;border-bottom:1px solid #00ffcc22;
}
/* Back buttons — red */
.screenBack{
  padding:5px 12px;font-size:12px;border-radius:7px;cursor:pointer;
  border:1px solid #ff4444aa;background:#2a0808;color:#ff6666;font-weight:bold;
  transition:background .15s;
}
.screenBack:hover{background:#3a1010;}

#ui{
  position:fixed;top:8px;left:50%;transform:translateX(-50%);
  text-align:center;font-size:16px;z-index:10;display:none;
  min-width:320px;pointer-events:none;
}
.rankBar{width:280px;height:16px;background:#333;border:2px solid #555;margin:3px auto;position:relative;overflow:hidden;border-radius:3px}
.rankFill{height:100%;background:linear-gradient(90deg,#00ff00,#ffff00);width:0%;transition:width .3s}
.rankText{position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);font-size:10px;color:#fff;font-weight:bold}
.rankLabel{font-size:22px;font-weight:bold;margin-bottom:4px;text-shadow:0 0 10px currentColor}
.xpLabel{font-size:13px;font-weight:bold;color:#00ffcc;margin:2px auto;text-shadow:0 0 8px #00ffcc}
.gemsLabel{font-size:15px;color:#00ffd0;font-weight:bold;margin:3px auto;text-shadow:0 0 8px #0ff}

#galaxyBanner{
  display:none;margin:6px 0 4px;padding:7px 12px;
  background:linear-gradient(90deg,#1a0030,#2d0055,#1a0030);
  border:1px solid #aa44cc;border-radius:8px;
  font-size:12px;font-weight:bold;color:#ee88ff;
  text-shadow:0 0 8px #cc44ff;letter-spacing:.5px;
  animation:nebulaPulse 2.5s ease-in-out infinite;
}
@keyframes nebulaPulse{0%,100%{box-shadow:0 0 6px #aa44cc44}50%{box-shadow:0 0 16px #cc44ffaa}}

#boostsGrid{display:flex;flex-wrap:wrap;justify-content:center;gap:7px;margin-top:6px;max-width:460px}
.boostCard{background:#0d1a2e;border:1px solid #334;border-radius:9px;padding:9px 10px;width:120px;text-align:center}
.boostCard.equipped{border-color:#00ffcc;box-shadow:0 0 8px #00ffcc44}
.boostCard .bn{font-size:13px;font-weight:bold;color:#fff;margin-bottom:3px}
.boostCard .bd{font-size:10px;color:#aaa;margin-bottom:6px;min-height:28px}
.boostCard .bc{font-size:11px;color:#00ffd0;margin-bottom:5px}
.boostCard button{
  padding:4px 10px;font-size:11px;border-radius:5px;cursor:pointer;font-weight:bold;
  border:none;background:linear-gradient(135deg,#003311,#004d1a);color:#00ff66;
  transition:filter .15s;
}
.boostCard button:hover{filter:brightness(1.2);}
.boostCard button:disabled{background:#1a1a1a;color:#444;cursor:not-allowed;filter:none;}
.boostCard button.unequip{background:linear-gradient(135deg,#2a0000,#440000);color:#ff6666;}
.boostCard button.unequip:hover{filter:brightness(1.2);}
#activeBoostsList{font-size:12px;color:#0ff;margin:4px 0 2px;min-height:16px}

#questsHeader{font-size:12px;color:#888;margin:4px 0 6px}
.questCard{background:#0d1520;border:1px solid #223;border-radius:10px;padding:9px 12px;margin:6px 0;text-align:left;position:relative}
.questCard.done{border-color:#00ff88;background:#091409}
.questCard .qt{font-size:13px;font-weight:bold;color:#fff;margin-bottom:3px}
.questCard .qd{font-size:11px;color:#aaa;margin-bottom:5px}
.questCard .qr{font-size:11px;color:#ffd700;position:absolute;top:9px;right:10px}
.questCard .qbar{height:8px;background:#1a1a2a;border-radius:4px;overflow:hidden;margin-bottom:3px}
.questCard .qfill{height:100%;border-radius:4px;transition:width .3s}
.questCard .qprog{font-size:10px;color:#888}
.questCard .qdone{font-size:12px;color:#00ff88;font-weight:bold}
.questRarity-common .qfill{background:#00cc88}.questRarity-rare .qfill{background:#4488ff}.questRarity-epic .qfill{background:#cc44ff}
.questRarity-common{border-left:3px solid #00cc88}.questRarity-rare{border-left:3px solid #4488ff}.questRarity-epic{border-left:3px solid #cc44ff}

#returnHomeBtn{display:none;margin:8px auto 0;padding:9px 26px;font-size:15px;border:2px solid #ff4444;background:#1a0000;color:#ff4444;border-radius:8px;cursor:pointer;font-weight:bold;transition:background .2s,color .2s;pointer-events:all}
#returnHomeBtn:hover{background:#ff4444;color:#fff}
#rankUpAnimation{display:none;position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);font-size:48px;font-weight:bold;color:gold;text-shadow:0 0 20px gold,0 0 40px gold;animation:rankUpPulse 1s ease-out;z-index:1000;pointer-events:none}
@keyframes rankUpPulse{0%{transform:translate(-50%,-50%) scale(1);opacity:1}100%{transform:translate(-50%,-150%) scale(1.5);opacity:0}}
#reviveFlash{display:none;position:fixed;left:0;top:0;width:100vw;height:100vh;background:rgba(0,255,255,.25);z-index:99999;pointer-events:none}
#gameOver{display:none;margin-top:5px;font-size:18px;color:#ff4444;pointer-events:all}
#godlyCooldown{position:fixed;left:50%;top:66px;transform:translateX(-50%);font-size:14px;font-weight:bold;color:#00fff7;background:rgba(0,0,0,.65);border-radius:6px;padding:2px 10px;display:none;z-index:999;box-shadow:0 0 8px #00fff7}
#soulBurstDisplay{position:fixed;left:50%;top:88px;transform:translateX(-50%);font-size:13px;font-weight:bold;color:#fffacd;background:rgba(0,0,0,.65);border-radius:6px;padding:2px 10px;display:none;z-index:999;box-shadow:0 0 10px #fffacd88}
#darkPulseDisplay{position:fixed;left:50%;top:88px;transform:translateX(-50%);font-size:13px;font-weight:bold;color:#dd44ff;background:rgba(0,0,0,.65);border-radius:6px;padding:2px 10px;display:none;z-index:999;box-shadow:0 0 10px #dd44ff88}
#laserDisplay{position:fixed;right:8px;top:8px;font-size:14px;font-weight:bold;color:#ff6644;text-shadow:0 0 8px #ff4400;display:none;z-index:999;pointer-events:none}
#teleportFlash{display:none;position:fixed;left:0;top:0;width:100vw;height:100vh;pointer-events:none;z-index:99998}
#toast{display:none;position:fixed;left:50%;bottom:40px;transform:translateX(-50%);background:rgba(0,0,0,.88);border:1px solid #00ffcc;border-radius:8px;padding:8px 18px;font-size:14px;font-weight:bold;color:#fff770;text-shadow:0 0 8px #0ff;z-index:9999;white-space:nowrap}
#controls-hint{position:fixed;bottom:10px;left:50%;transform:translateX(-50%);font-size:11px;color:#334;z-index:999;pointer-events:none}
#singularityDisplay{position:fixed;left:50%;top:110px;transform:translateX(-50%);font-size:13px;font-weight:bold;color:#ff6666;background:rgba(0,0,0,.65);border-radius:6px;padding:2px 10px;display:none;z-index:999;box-shadow:0 0 10px #ff666688}
#chronosDisplay{position:fixed;left:50%;top:110px;transform:translateX(-50%);font-size:13px;font-weight:bold;color:#c8c8ff;background:rgba(0,0,0,.65);border-radius:6px;padding:2px 10px;display:none;z-index:999;box-shadow:0 0 10px #aaaaff88}
#galaxyHUD{position:fixed;left:8px;top:8px;font-size:11px;font-weight:bold;color:#ee88ff;background:rgba(20,0,40,.75);border:1px solid #aa44cc44;border-radius:6px;padding:3px 9px;display:none;z-index:999;pointer-events:none;text-shadow:0 0 6px #dd44ff}
@keyframes teleportBurst{0%{opacity:.85;transform:scale(1)}60%{opacity:.4;transform:scale(1.08)}100%{opacity:0;transform:scale(1.18)}}
.teleport-anim{animation:teleportBurst .28s ease-out forwards}

#mobile-btns{position:fixed;bottom:16px;left:0;right:0;display:none;z-index:200;pointer-events:none;padding:0 12px}
#mobile-btns-row{display:flex;align-items:center;justify-content:space-between;max-width:400px;margin:0 auto}
.mbtn{width:68px;height:68px;border-radius:50%;border:2px solid rgba(0,255,204,0.45);background:rgba(0,0,0,0.6);color:#00ffcc;font-size:28px;display:flex;align-items:center;justify-content:center;pointer-events:all;touch-action:manipulation;user-select:none;-webkit-user-select:none;cursor:pointer;-webkit-tap-highlight-color:transparent;transition:background .08s,border-color .08s}
.mbtn:active{background:rgba(0,255,204,0.2);border-color:#00ffcc}
#mbtn-boost{background:rgba(0,0,0,0.5);border-color:rgba(255,255,255,0.25);color:#fff;font-size:22px}
#mbtn-boost:active{background:rgba(255,255,255,0.12)}
#mbtn-ability{border-color:rgba(255,200,0,0.5);color:#ffdd00;font-size:22px}
#mbtn-ability:active{background:rgba(255,200,0,0.18)}
#mobile-toggle-row{margin:8px 0 2px;display:flex;align-items:center;justify-content:center;gap:8px;font-size:12px;color:#666}
#mobile-toggle{padding:4px 12px;font-size:12px;font-weight:bold;border-radius:6px;cursor:pointer;border:1px solid #444;background:#1a1a2e;color:#777;transition:all .15s}
#mobile-toggle.on{border-color:#00ffcc;color:#00ffcc;background:#0d2137}

/* ── TUTORIAL ── */
#tutorialOverlay{position:fixed;inset:0;z-index:9000;background:rgba(0,0,0,.72);display:flex;align-items:center;justify-content:center;}
#tutorialBox{background:linear-gradient(160deg,#040d1e,#071428);border:1px solid #00ffcc55;border-radius:16px;padding:22px 24px 18px;width:min(380px,92vw);text-align:center;box-shadow:0 0 40px #00ffcc22,0 0 80px #0008;animation:tutSlideIn .35s ease-out;position:relative;}
@keyframes tutSlideIn{from{opacity:0;transform:translateY(18px)}to{opacity:1;transform:translateY(0)}}
#tutorialProgress{margin-bottom:12px}
.tut-dot{display:inline-block;width:8px;height:8px;border-radius:50%;background:#223;margin:0 3px;transition:background .2s}
.tut-dot.active{background:#00ffcc;box-shadow:0 0 6px #00ffcc}
.tut-dot.done{background:#00886655}
#tutorialIcon{font-size:44px;margin-bottom:8px;line-height:1}
#tutorialTitle{font-size:18px;font-weight:bold;color:#00ffcc;margin-bottom:8px;text-shadow:0 0 10px #00ffcc66}
#tutorialBody{font-size:13px;color:#b0c8d8;line-height:1.65;min-height:60px;margin-bottom:16px}
#tutorialBody strong{color:#fff}
#tutorialBody .key{display:inline-block;background:#0a1a2a;border:1px solid #00ffcc55;border-radius:5px;padding:1px 7px;font-size:12px;color:#00ffcc;font-family:monospace;margin:0 2px;}
#tutorialBtns{display:flex;gap:10px;justify-content:center}
#tutorialSkip{padding:7px 16px;font-size:12px;border-radius:7px;cursor:pointer;border:1px solid #334;background:transparent;color:#556;transition:color .15s,border-color .15s;}
#tutorialSkip:hover{color:#aaa;border-color:#556}
#tutorialNext{padding:8px 22px;font-size:14px;font-weight:bold;border-radius:7px;cursor:pointer;border:2px solid #00ffcc;background:#00ffcc18;color:#00ffcc;transition:background .15s;}
#tutorialNext:hover{background:#00ffcc33}
#tutorialNext.finish{border:3px solid #b8860b;background:linear-gradient(180deg,#ffe135 0%,#ffc200 45%,#e89900 100%);color:#2a1a00;box-shadow:0 3px 0 #8a6000;text-shadow:0 1px 0 rgba(255,255,255,.3);}
#tutorialNext.finish:hover{filter:brightness(1.08);}
.tut-arrow{position:fixed;pointer-events:none;z-index:8999;font-size:28px;animation:tutBounce .7s ease-in-out infinite alternate;}
@keyframes tutBounce{from{transform:translateY(0)}to{transform:translateY(-8px)}}
</style>
</head>
<body>
<div id="renderer-container"></div>
<div id="reviveFlash"></div>
<div id="teleportFlash"></div>
<div id="rankUpAnimation">⭐ RANK UP ⭐</div>
<div id="toast"></div>
<div id="godlyCooldown"></div>
<div id="soulBurstDisplay">✨ Soul Burst: <span id="soulBurstCount">20</span> dodges</div>
<div id="darkPulseDisplay">💜 Dark Pulse: <span id="darkPulseCount">15</span></div>
<div id="laserDisplay">🔥 Laser: <span id="laserLeft">3</span></div>
<div id="singularityDisplay">🌀 Singularity: <span id="singularityTimer">15</span>s</div>
<div id="chronosDisplay">🕰 Chronos: <span id="chronosLabel">Solid 8s</span></div>
<div id="galaxyHUD">🌌 NEBULA RIFT</div>
<div id="controls-hint">← → / A D = Move &nbsp;|&nbsp; SPACE = Boost &nbsp;|&nbsp; L = Laser &nbsp;|&nbsp; F = Ability &nbsp;|&nbsp; R = Home</div>

<div id="tutorialOverlay" style="display:none">
  <div id="tutorialBox">
    <div id="tutorialProgress"><span id="tutorialDots"></span></div>
    <div id="tutorialIcon"></div>
    <div id="tutorialTitle"></div>
    <div id="tutorialBody"></div>
    <div id="tutorialBtns">
      <button id="tutorialSkip">Skip</button>
      <button id="tutorialNext">Next →</button>
    </div>
  </div>
</div>

<div id="mobile-btns">
  <div id="mobile-btns-row">
    <div class="mbtn" id="mbtn-left">◀</div>
    <div class="mbtn" id="mbtn-boost">BOOST</div>
    <div class="mbtn" id="mbtn-ability" style="display:none">⚡</div>
    <div class="mbtn" id="mbtn-right">▶</div>
  </div>
</div>

<div id="homeMain">
  <div id="hmTop">
    <div id="hmPilot"><input id="playerNameInput" type="text" maxlength="16" placeholder="Pilot name…"/></div>
    <div id="hmRankBlock">
      <div class="rankLabel" id="homeRank" style="color:#A0A0A0">CADET</div>
      <div class="rankBar" style="width:200px"><div class="rankFill" id="homeRankFill"></div><div class="rankText" id="homeRankText">0 / 100 XP</div></div>
      <div id="homeXpLabel" class="xpLabel">XP: 0</div>
    </div>
    <div id="hmGems" class="gemsLabel">💎 <span class="gemCount">0</span></div>
  </div>
  <div id="galaxyBanner">🌌 NEBULA RIFT GALAXY UNLOCKED — 3 new ships available!</div>
  <div id="hmShipCard">
    <div id="hmShipSvg"></div>
    <div id="hmShipInfo">
      <div id="hmShipName"></div>
      <div id="hmShipPower"></div>
    </div>
  </div>
  <button id="startBtn">▶ PLAY</button>
  <div id="hmNav">
    <button class="hmNavBtn" id="navHangar">🚀 Ships</button>
    <button class="hmNavBtn" id="navBoosts">⚡ Boosts</button>
    <button class="hmNavBtn" id="navQuests">📜 Quests</button>
    <button class="hmNavBtn" id="navShop">🛒 Shop</button>
    <button class="hmNavBtn" id="navPass">✦ StellarPass</button>
  </div>
  <div id="mobile-toggle-row">
    <span>📱 Mobile Controls</span>
    <button id="mobile-toggle">OFF</button>
  </div>
</div>

<div id="hangarScreen" style="display:none">
  <div id="hangarHeader">
    <button id="hangarBack">← Back</button>
    <div id="hangarTitle">🚀 Ship Hangar</div>
    <button id="boosterBtn">🎲 Booster (1💎)</button>
  </div>
  <div id="hangarBody">
    <div id="hangarList"></div>
    <div id="hangarDetail">
      <canvas id="shipPreviewCanvas"></canvas>
      <div id="hdName"></div>
      <div id="hdGalaxy"></div>
      <div id="hdPower"></div>
      <div id="hdPowerDesc"></div>
      <div id="hdUnlock"></div>
      <div id="hdBoostDesc"></div>
      <div id="hdSkinInfo"></div>
      <button id="hdSelectBtn">✓ Select Ship</button>
    </div>
  </div>
</div>

<div id="boostsScreen" style="display:none">
  <div id="boostsScreenHeader">
    <button class="screenBack">← Back</button>
    <div style="font-size:16px;font-weight:bold;color:#ff6600">⚡ Run Boosts</div>
    <div style="font-size:12px;color:#888">💎 <span class="gemCount">0</span></div>
  </div>
  <div style="font-size:11px;color:#666;margin:4px 0 8px">Equip before a run. Costs gems. Active for one run.</div>
  <div id="activeBoostsList"></div>
  <div id="boostsGrid"></div>
</div>

<div id="questsScreen" style="display:none">
  <div id="questsScreenHeader">
    <button class="screenBack">← Back</button>
    <div style="font-size:16px;font-weight:bold;color:#8844dd">📜 Quests</div>
    <div style="font-size:12px;color:#888">💎 <span class="gemCount">0</span></div>
  </div>
  <div id="questsHeader"></div>
  <div id="questsList"></div>
</div>

<!-- SHOP SCREEN -->
<div id="shopScreen" style="display:none">
  <div id="shopHeader">
    <button class="screenBack">← Back</button>
    <div id="shopTitle">🛒 Skin Shop</div>
    <div id="shopTimerBox">
      <span class="timerIcon">⏱</span>
      <span>Refreshes in</span>
      <span class="timerVal" id="shopTimerVal">20:00</span>
    </div>
  </div>
  <div id="shopBody">
    <div id="shopSubtitle">3 skins available · Buy to keep forever · Equip any owned skin in the Hangar</div>
    <div id="shopCards"></div>
  </div>
</div>

<!-- STELLAR PASS SCREEN -->
<div id="passScreen" style="display:none">
  <div id="passHeader">
    <button class="screenBack">← Back</button>
    <div id="passTitle">✦ STELLAR PASS</div>
    <div id="passGemCount">💎 <span class="gemCount">0</span></div>
  </div>
  <div id="passBody">
    <div id="passBanner">
      <div id="passBannerTitle">✦ STELLAR PASS ✦</div>
      <div id="passBannerSub">Earn Pass XP by playing runs · Claim rewards as you level up</div>
    </div>
    <div id="passXpBar">
      <div id="passXpLabel"><span>Pass XP: <strong id="passXpCur">0</strong></span><span id="passXpNext"></span></div>
      <div id="passXpTrack"><div id="passXpFill" style="width:0%"></div></div>
    </div>
    <div id="passTiersGrid"></div>
    <div id="passFooter">
      Each run earns Pass XP equal to your score ÷ 10 &nbsp;·&nbsp; All rewards are yours forever<br>
      🌈 <strong style="animation:chromaticShift 2.5s linear infinite;display:inline-block">Chromatic</strong> — the rarest tier, with shifting rainbow colors
    </div>
  </div>
</div>

<div id="ui">
  <div class="rankLabel" id="rankDisplay" style="color:#A0A0A0">CADET</div>
  <div class="rankBar"><div class="rankFill" id="gameRankFill"></div><div class="rankText" id="gameRankText">0 / 100 XP</div></div>
  <div class="xpLabel" id="xpDisplay">XP: 0</div>
  <div class="gemsLabel">💎 <span class="gemCount">0</span></div>
  <div>Score: <span id="score">0</span> &nbsp;|&nbsp; Best: <span id="highScore">0</span></div>
  <div id="gameOver">💥 GAME OVER</div>
  <button id="returnHomeBtn">🏠 Return Home</button>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
const XP_RANKS=[
  {name:"Cadet",xp:0,color:"#A0A0A0"},{name:"Pilot",xp:100,color:"#00FF00"},
  {name:"Commander",xp:300,color:"#0099FF"},{name:"Captain",xp:600,color:"#FF6600"},
  {name:"Admiral",xp:1200,color:"#FF00FF"},{name:"Legend",xp:2500,color:"#FFFF00"},
  {name:"Godly",xp:4000,color:"#00fff7"},{name:"Mythic",xp:6000,color:"#ff77ff"},
  {name:"Eternal",xp:9000,color:"#fffacd"},{name:"Cosmic",xp:13000,color:"#ff6666"},
  {name:"Transcendent",xp:20000,color:"#ffffff"},{name:"Nova",xp:25000,color:"#ff44bb"},
  {name:"Rift Walker",xp:40000,color:"#00ffaa"},{name:"Omega",xp:65000,color:"#ff8800"},
];
const COSMETICS=[
  {id:0,name:"Cadet Viper",color:"#00ffcc",unlockXp:0,galaxy:1,powerName:"Standard",powerDesc:"No special power."},
  {id:1,name:"Pilot Falcon",color:"#00FF00",unlockXp:100,galaxy:1,powerName:"Double Boost",powerDesc:"Spacebar boost is twice as fast."},
  {id:2,name:"Commander Nova",color:"#0099FF",unlockXp:300,galaxy:1,powerName:"Shield on Dodge",powerDesc:"Gain a shield every 5 asteroids dodged."},
  {id:3,name:"Captain Phoenix",color:"#FF6600",unlockXp:600,galaxy:1,powerName:"Milestone XP+",powerDesc:"+50% XP for score milestones."},
  {id:4,name:"Admiral Eclipse",color:"#FF00FF",unlockXp:1200,galaxy:1,powerName:"Slow Asteroids",powerDesc:"Asteroid speed −20%."},
  {id:5,name:"Legend Starcruiser",color:"#FFFF00",unlockXp:2500,galaxy:1,powerName:"Revive",powerDesc:"Survive first crash once per game."},
  {id:6,name:"Godly Dragon",color:"#00fff7",unlockXp:4000,galaxy:1,powerName:"Time Annihilate",powerDesc:"Press F every 30s: destroy all asteroids + freeze 3s."},
  {id:7,name:"Mythic Phantom",color:"#ff77ff",unlockXp:6000,galaxy:1,powerName:"Void Teleport",powerDesc:"Instantly teleport between lanes — no sliding. Teleport through asteroids harmlessly!"},
  {id:8,name:"Eternal Seraph",color:"#fffacd",unlockXp:9000,galaxy:1,powerName:"Soul Burst",powerDesc:"Every 20 asteroids dodged, your soul erupts — all asteroids annihilated + burst of XP."},
  {id:9,name:"Cosmic Harbinger",color:"#ff6666",unlockXp:13000,galaxy:1,powerName:"Singularity",powerDesc:"Gems drawn toward you passively. Every 15s a gravity pulse pulls ALL gems to your lane."},
  {id:10,name:"Transcendent Aeon",color:"#c8c8ff",unlockXp:20000,galaxy:1,powerName:"Chronos Shift",powerDesc:"Every 8s you phase into another dimension for 3s — asteroids pass through you harmlessly."},
  {id:11,name:"Nebula Wraith",color:"#dd44ff",unlockXp:25000,galaxy:2,powerName:"Dark Pulse",powerDesc:"Every 15 asteroids dodged, your lane erupts — all asteroids in your lane destroyed + gain a shield."},
  {id:12,name:"Void Colossus",color:"#00ffaa",unlockXp:40000,galaxy:2,powerName:"Iron Tide",powerDesc:"Asteroids in adjacent lanes move 25% slower. Gain a shield every 20 asteroids dodged."},
  {id:13,name:"Omega Prime",color:"#ff8800",unlockXp:65000,galaxy:2,powerName:"Cosmic Reckoning",powerDesc:"Press F every 20s: destroy ALL asteroids, freeze 5s, and earn up to 5 💎."},
  {id:14,name:"Stellar Sovereign",color:"#cc44ff",unlockXp:0,galaxy:0,powerName:"Chromatic Surge",powerDesc:"Every 10 asteroids dodged, a random power activates: shield, freeze, destroy lane, or gem pull. StellarPass exclusive.",stellarPassOnly:true},
];
const RUN_BOOSTS=[
  {id:"hyperdrive",name:"⚡ Hyperdrive",desc:"Move 2× faster between lanes",cost:1},
  {id:"iron_shell",name:"🛡️ Iron Shell",desc:"Start the run with a shield",cost:1},
  {id:"xp_surge",name:"✨ XP Surge",desc:"Earn 3× XP this entire run",cost:2},
  {id:"null_field",name:"❄️ Null Field",desc:"Asteroids 35% slower this run",cost:2},
  {id:"laser_shot",name:"🔥 Laser Shot",desc:"Press L to blast your lane (3 shots)",cost:2},
];
const QUEST_TIERS=[
  {targets:[100,150,200],gems:1},{targets:[200,250,300],gems:1},{targets:[300,400,500],gems:2},
  {targets:[500,600,750],gems:2},{targets:[750,900,1100],gems:3},{targets:[1000,1250,1500],gems:3},
  {targets:[1500,1750,2000],gems:4},{targets:[2000,2500,3000],gems:4},{targets:[3000,4000,5000],gems:5},
];
const SKINS=[
  {id:0, shipId:0,  name:"Midnight Viper",   color:"#ff3366", rarity:"rare",      cost:2},
  {id:1, shipId:1,  name:"Solar Falcon",     color:"#ff9900", rarity:"rare",      cost:2},
  {id:2, shipId:2,  name:"Crimson Nova",     color:"#ff0066", rarity:"epic",      cost:3},
  {id:3, shipId:3,  name:"Frost Phoenix",    color:"#00ccff", rarity:"rare",      cost:2},
  {id:4, shipId:4,  name:"Shadow Eclipse",   color:"#00ff88", rarity:"epic",      cost:3},
  {id:5, shipId:5,  name:"Neon Cruiser",     color:"#ff00aa", rarity:"epic",      cost:3},
  {id:6, shipId:6,  name:"Inferno Dragon",   color:"#ff4400", rarity:"epic",      cost:3},
  {id:7, shipId:7,  name:"Azure Phantom",    color:"#00ffff", rarity:"epic",      cost:3},
  {id:8, shipId:8,  name:"Dark Seraph",      color:"#cc44ff", rarity:"legendary", cost:4},
  {id:9, shipId:9,  name:"Jade Harbinger",   color:"#44ff88", rarity:"legendary", cost:4},
  {id:10,shipId:10, name:"Ember Aeon",       color:"#ff8844", rarity:"legendary", cost:4},
  {id:11,shipId:11, name:"Toxic Wraith",     color:"#88ff00", rarity:"legendary", cost:4},
  {id:12,shipId:12, name:"Blood Colossus",   color:"#ff2222", rarity:"mythic",    cost:5},
  {id:13,shipId:13, name:"Ice Omega",        color:"#00ddff", rarity:"mythic",    cost:5},
  {id:14,shipId:0,  name:"Chromatic Viper",  color:"#cc44ff", rarity:"chromatic", cost:0, passOnly:true},
  {id:15,shipId:7,  name:"Chromatic Phantom",color:"#ff44bb", rarity:"chromatic", cost:0, passOnly:true},
];

// THREE.JS
const scene=new THREE.Scene();
scene.background=new THREE.Color(0x000510);
scene.fog=new THREE.FogExp2(0x000510,0.008);
const renderer=new THREE.WebGLRenderer({antialias:true,powerPreference:'high-performance'});
renderer.setPixelRatio(Math.min(window.devicePixelRatio,2));
renderer.setSize(window.innerWidth,window.innerHeight);
renderer.shadowMap.enabled=true;
document.getElementById("renderer-container").appendChild(renderer.domElement);
const camera=new THREE.PerspectiveCamera(72,window.innerWidth/window.innerHeight,0.1,600);
camera.position.set(0,6,12);camera.lookAt(0,0,-20);
window.addEventListener("resize",()=>{renderer.setSize(window.innerWidth,window.innerHeight);camera.aspect=window.innerWidth/window.innerHeight;camera.updateProjectionMatrix();});
const ambientLight=new THREE.AmbientLight(0x111233,2);scene.add(ambientLight);
const sunLight=new THREE.DirectionalLight(0x6688cc,1.5);sunLight.position.set(-5,10,5);scene.add(sunLight);
const rimLight=new THREE.DirectionalLight(0x00ffcc,0.5);rimLight.position.set(5,-5,-10);scene.add(rimLight);
const shipPointLight=new THREE.PointLight(0xffffff,2,12);shipPointLight.position.set(0,3,2);scene.add(shipPointLight);
const engineLight=new THREE.PointLight(0x00ffcc,3,6);scene.add(engineLight);
const voidLight=new THREE.PointLight(0xff44ff,0,14);scene.add(voidLight);
const soulLight=new THREE.PointLight(0xfffacd,0,20);scene.add(soulLight);
const singLight=new THREE.PointLight(0xff3333,0,16);scene.add(singLight);
const chronosLight=new THREE.PointLight(0x8888ff,0,16);scene.add(chronosLight);
const nebulaLight=new THREE.PointLight(0xff44bb,0,120);nebulaLight.position.set(0,25,-90);scene.add(nebulaLight);

const LANES_3D=[-5,0,5],SPAWN_Z=-130,PASS_Z=5,SPEED_SCALE=0.175;
const XP_PER_ASTEROID=10,XP_MILESTONE_STEP=100,XP_PER_MILESTONE=40,XP_PASSIVE_SEC=1;

let shipGroup=null,shieldMesh=null,laserBeam=null;
let engineExhaust=[],particles3D=[],asteroids=[],stars3D=null;
let seraphHaloRing=null,seraphWingGlow=0,seraphSoulBurstFlashFrames=0;
let ghostDecoy=null,ghostDecoyLane=-1,ghostDecoyFrames=0;
let isTeleporting=false,teleportFrames=0;
const TELEPORT_DURATION=8;
let voidParticles=[],soulParticles=[],gemObjects=[];
let chronosGhostMesh=null,chronosPhased=false,chronosTimer=0;
const CHRONOS_SOLID=480,CHRONOS_PHASED=180;
let singularityTimer=0;
const SINGULARITY_INTERVAL=900;
let currentLaneGroup=null,nebulaGroup=null;

// STARFIELD
function createStarfield(){
  const geo=new THREE.BufferGeometry(),positions=[],colors=[];
  for(let i=0;i<3000;i++){
    positions.push((Math.random()-.5)*250,(Math.random()-.5)*140,Math.random()*-250);
    const b=0.5+Math.random()*0.5;colors.push(b,b,Math.min(1,b+Math.random()*0.3));
  }
  geo.setAttribute("position",new THREE.Float32BufferAttribute(positions,3));
  geo.setAttribute("color",new THREE.Float32BufferAttribute(colors,3));
  return new THREE.Points(geo,new THREE.PointsMaterial({size:0.35,sizeAttenuation:true,vertexColors:true,transparent:true,opacity:0.9}));
}
stars3D=createStarfield();scene.add(stars3D);
function setStarfieldColors(g2){
  const col=stars3D.geometry.attributes.color.array;
  for(let i=0;i<col.length;i+=3){
    if(g2){const r=0.55+Math.random()*0.45;col[i]=r;col[i+1]=Math.random()*0.18;col[i+2]=r*(0.6+Math.random()*0.4);}
    else{const b=0.5+Math.random()*0.5;col[i]=b;col[i+1]=b;col[i+2]=Math.min(1,b+Math.random()*0.3);}
  }
  stars3D.geometry.attributes.color.needsUpdate=true;
}
function updateStarfield(speed2d){
  const s=speed2d*SPEED_SCALE*1.8,pos=stars3D.geometry.attributes.position.array;
  for(let i=0;i<pos.length;i+=3){pos[i+2]+=s;if(pos[i+2]>80){pos[i]=(Math.random()-.5)*250;pos[i+1]=(Math.random()-.5)*140;pos[i+2]=-250+Math.random()*20;}}
  stars3D.geometry.attributes.position.needsUpdate=true;
}

// LANE SCENE
function createLaneScene(g2=false){
  const grp=new THREE.Group();
  const floorCol=g2?0x0e0018:0x010815,gridCol=g2?0x1a0030:0x001830,div1Col=g2?0x770099:0x004488,div2Col=g2?0xbb55dd:0x0088ff,outerCol=g2?0x440066:0x002244,laneCol=g2?0x330055:0x003355,horizCol=g2?0x660099:0x0044aa;
  const floor=new THREE.Mesh(new THREE.PlaneGeometry(22,300),new THREE.MeshBasicMaterial({color:floorCol,side:THREE.DoubleSide,transparent:true,opacity:0.8}));
  floor.rotation.x=Math.PI/2;floor.position.set(0,-1.8,-100);grp.add(floor);
  for(let z=-130;z<=10;z+=12){const g=new THREE.BufferGeometry().setFromPoints([new THREE.Vector3(-11,-1.8,z),new THREE.Vector3(11,-1.8,z)]);grp.add(new THREE.Line(g,new THREE.LineBasicMaterial({color:gridCol,transparent:true,opacity:0.4})));}
  [-2.5,2.5].forEach(x=>{const pts=[new THREE.Vector3(x,-1.8,-130),new THREE.Vector3(x,-1.8,10)];const g=new THREE.BufferGeometry().setFromPoints(pts);grp.add(new THREE.Line(g,new THREE.LineBasicMaterial({color:div1Col,transparent:true,opacity:0.7})));grp.add(new THREE.Line(g,new THREE.LineBasicMaterial({color:div2Col,transparent:true,opacity:0.25})));});
  [-7.5,7.5].forEach(x=>{const pts=[new THREE.Vector3(x,-1.8,-130),new THREE.Vector3(x,-1.8,10)];const g=new THREE.BufferGeometry().setFromPoints(pts);grp.add(new THREE.Line(g,new THREE.LineBasicMaterial({color:outerCol,transparent:true,opacity:0.5})));});
  LANES_3D.forEach(x=>{const pts=[new THREE.Vector3(x,-1.8,-130),new THREE.Vector3(x,-1.8,10)];const g=new THREE.BufferGeometry().setFromPoints(pts);grp.add(new THREE.Line(g,new THREE.LineBasicMaterial({color:laneCol,transparent:true,opacity:0.3})));});
  const horizon=new THREE.Mesh(new THREE.SphereGeometry(4,16,16),new THREE.MeshBasicMaterial({color:horizCol,transparent:true,opacity:0.12,side:THREE.BackSide}));
  horizon.position.set(0,0,-140);grp.add(horizon);
  return grp;
}
function switchLaneScene(g2){if(currentLaneGroup)scene.remove(currentLaneGroup);currentLaneGroup=createLaneScene(g2);scene.add(currentLaneGroup);}
function buildNebulaGroup(){
  if(nebulaGroup){scene.remove(nebulaGroup);nebulaGroup=null;}
  nebulaGroup=new THREE.Group();
  const cols=[0x330044,0x440022,0x220033,0x441133,0x331155,0x220044];
  for(let i=0;i<12;i++){const m=new THREE.Mesh(new THREE.SphereGeometry(16+Math.random()*28,6,4),new THREE.MeshBasicMaterial({color:cols[i%cols.length],transparent:true,opacity:0.03+Math.random()*0.07,side:THREE.FrontSide}));m.position.set((Math.random()-.5)*200,(Math.random()-.5)*80,-65-Math.random()*130);nebulaGroup.add(m);}
  for(let i=0;i<5;i++){const m=new THREE.Mesh(new THREE.TorusGeometry(30+Math.random()*40,0.8+Math.random()*1.5,5,24),new THREE.MeshBasicMaterial({color:0x660099,transparent:true,opacity:0.03+Math.random()*0.05}));m.rotation.set(Math.random()*Math.PI,Math.random()*Math.PI,Math.random()*Math.PI);m.position.set((Math.random()-.5)*160,(Math.random()-.5)*60,-80-Math.random()*100);nebulaGroup.add(m);}
  scene.add(nebulaGroup);
}
function applyGalaxyVisuals(g2){
  if(g2){scene.background.set(0x090015);scene.fog.color.set(0x090015);scene.fog.density=0.007;ambientLight.color.set(0x1a0822);ambientLight.intensity=2.3;sunLight.color.set(0x9933cc);sunLight.intensity=1.1;rimLight.color.set(0xff44bb);rimLight.intensity=0.9;nebulaLight.intensity=0.8;buildNebulaGroup();if(nebulaGroup)nebulaGroup.visible=true;}
  else{scene.background.set(0x000510);scene.fog.color.set(0x000510);scene.fog.density=0.008;ambientLight.color.set(0x111233);ambientLight.intensity=2;sunLight.color.set(0x6688cc);sunLight.intensity=1.5;rimLight.color.set(0x00ffcc);rimLight.intensity=0.5;nebulaLight.intensity=0;if(nebulaGroup)nebulaGroup.visible=false;}
  switchLaneScene(g2);setStarfieldColors(g2);
}

// SHIP MESH
function getShipColor(id){
  if(id===14){
    // Sovereign uses cycling hue at current time
    const h=(Date.now()*0.0003)%1;
    const c=new THREE.Color();c.setHSL(h,1,0.65);return '#'+c.getHexString();
  }
  const equipped=getEquippedSkin(id);
  if(equipped!==null){const sk=SKINS.find(s=>s.id===equipped);if(sk)return sk.color;}
  return COSMETICS[id].color;
}
function makeMat(hex,e=0.35,m=0.6,r=0.25){const c=new THREE.Color(hex);return new THREE.MeshStandardMaterial({color:c,emissive:c.clone().multiplyScalar(e),metalness:m,roughness:r});}
function createShipMesh(id){
  const group=new THREE.Group(),col=getShipColor(id),mat=makeMat(col);
  seraphHaloRing=null;chronosGhostMesh=null;group.userData.wraithOrbs=[];
  if(id===0){const body=new THREE.Mesh(new THREE.ConeGeometry(0.75,3.5,5),mat);body.rotation.x=-Math.PI/2;body.position.z=-0.3;group.add(body);const finGeo=new THREE.BoxGeometry(0.12,1.1,1.4);[-1,1].forEach(s=>{const fin=new THREE.Mesh(finGeo,mat.clone());fin.position.set(s*1.1,-0.3,1.0);group.add(fin);});const cockpit=new THREE.Mesh(new THREE.SphereGeometry(0.35,6,6),makeMat(col,0.8,0.3,0.1));cockpit.position.set(0,0.3,-0.5);group.add(cockpit);}
  else if(id===1){const body=new THREE.Mesh(new THREE.ConeGeometry(0.65,4,4),mat);body.rotation.x=-Math.PI/2;body.position.z=-0.2;group.add(body);const wS=new THREE.Shape();wS.moveTo(0,0);wS.lineTo(3.5,0.5);wS.lineTo(2.8,2.5);wS.lineTo(0,1.5);const wG=new THREE.ShapeGeometry(wS);const wM=new THREE.MeshStandardMaterial({color:new THREE.Color(col),emissive:new THREE.Color(col).multiplyScalar(0.3),side:THREE.DoubleSide,metalness:0.5,roughness:0.3});[-1,1].forEach(s=>{const w=new THREE.Mesh(wG,wM.clone());w.rotation.x=Math.PI/2;w.position.set(s*0.65,-0.1,0.5);if(s===-1){w.rotation.y=Math.PI;w.position.x=-0.65;}w.scale.x=s;group.add(w);});}
  else if(id===2){const body=new THREE.Mesh(new THREE.SphereGeometry(1,10,8),mat);body.scale.set(0.9,0.7,2.2);group.add(body);const nG=new THREE.SphereGeometry(0.38,6,6);[-1.3,1.3].forEach(x=>{const n=new THREE.Mesh(nG,mat.clone());n.position.set(x,0,0.6);group.add(n);});const nose=new THREE.Mesh(new THREE.ConeGeometry(0.45,1.8,6),mat.clone());nose.rotation.x=-Math.PI/2;nose.position.set(0,0,-2.5);group.add(nose);}
  else if(id===3){const body=new THREE.Mesh(new THREE.OctahedronGeometry(1.6,0),mat);body.scale.set(0.8,0.55,2.2);group.add(body);const tail=new THREE.Mesh(new THREE.ConeGeometry(0.5,1.8,4),mat.clone());tail.rotation.x=Math.PI/2;tail.position.set(0,0,2.2);group.add(tail);const wing=new THREE.Mesh(new THREE.BoxGeometry(3.2,0.12,1.2),mat.clone());wing.position.set(0,0,0.4);group.add(wing);}
  else if(id===4){const disc=new THREE.Mesh(new THREE.CylinderGeometry(1.8,1.4,0.6,12),mat);disc.rotation.x=Math.PI/2;group.add(disc);const dome=new THREE.Mesh(new THREE.SphereGeometry(0.9,8,6,0,Math.PI*2,0,Math.PI/2),mat.clone());group.add(dome);const ring=new THREE.Mesh(new THREE.TorusGeometry(2.4,0.18,7,28),mat.clone());ring.rotation.x=Math.PI/2;group.add(ring);const nose2=new THREE.Mesh(new THREE.ConeGeometry(0.3,1.2,5),mat.clone());nose2.rotation.x=-Math.PI/2;nose2.position.set(0,0,-1.9);group.add(nose2);}
  else if(id===5){const body=new THREE.Mesh(new THREE.BoxGeometry(2.0,0.85,3.5),mat);group.add(body);const n=new THREE.Mesh(new THREE.ConeGeometry(1.05,2,4),mat.clone());n.rotation.x=-Math.PI/2;n.position.set(0,0,-2.6);group.add(n);const blade=new THREE.Mesh(new THREE.BoxGeometry(4.5,0.12,1.8),mat.clone());blade.position.set(0,0,0.3);group.add(blade);const gun=new THREE.Mesh(new THREE.CylinderGeometry(0.15,0.15,1.5,6),mat.clone());gun.rotation.x=Math.PI/2;gun.position.set(0,0.5,-1.8);group.add(gun);}
  else if(id===6){const body=new THREE.Mesh(new THREE.CylinderGeometry(0.65,1.2,4,7),mat);body.rotation.x=Math.PI/2;group.add(body);const neck=new THREE.Mesh(new THREE.ConeGeometry(0.5,2.5,6),mat.clone());neck.rotation.x=-Math.PI/2;neck.position.set(0,0.3,-2.8);group.add(neck);const wS=new THREE.Shape();wS.moveTo(0,0);wS.lineTo(3.2,-0.8);wS.lineTo(2.2,2.5);wS.lineTo(0.8,2.8);wS.lineTo(0,1.5);const wM=new THREE.MeshStandardMaterial({color:new THREE.Color(col),emissive:new THREE.Color(col).multiplyScalar(0.4),side:THREE.DoubleSide,metalness:0.3,roughness:0.4,transparent:true,opacity:0.92});[-1,1].forEach(s=>{const w=new THREE.Mesh(new THREE.ShapeGeometry(wS),wM.clone());w.rotation.x=Math.PI/2;w.scale.x=s;w.position.set(s*0.65,-0.2,0.5);group.add(w);});}
  else if(id===7){const hull=new THREE.Mesh(new THREE.ConeGeometry(1.1,4.5,3),mat);hull.rotation.x=-Math.PI/2;hull.position.z=-0.2;group.add(hull);const vwS=new THREE.Shape();vwS.moveTo(0,0);vwS.lineTo(3.8,0.2);vwS.lineTo(3.0,1.6);vwS.lineTo(1.2,2.2);vwS.lineTo(0,1.4);const vwM=new THREE.MeshStandardMaterial({color:new THREE.Color(col),emissive:new THREE.Color(col).multiplyScalar(0.55),side:THREE.DoubleSide,metalness:0.2,roughness:0.3,transparent:true,opacity:0.88});[-1,1].forEach(s=>{const vw=new THREE.Mesh(new THREE.ShapeGeometry(vwS),vwM.clone());vw.rotation.x=Math.PI/2;vw.scale.x=s;vw.position.set(s*0.55,-0.05,0.6);group.add(vw);});const orb=new THREE.Mesh(new THREE.SphereGeometry(0.42,10,10),new THREE.MeshStandardMaterial({color:0x220033,emissive:new THREE.Color(col),emissiveIntensity:1.2,metalness:0,roughness:0,transparent:true,opacity:0.95}));orb.position.set(0,0.22,-0.6);group.add(orb);const vR=new THREE.Mesh(new THREE.TorusGeometry(0.62,0.06,7,22),new THREE.MeshBasicMaterial({color:new THREE.Color(col),transparent:true,opacity:0.8}));vR.rotation.x=Math.PI/2;vR.position.set(0,0.22,-0.6);group.add(vR);}
  else if(id===8){const wG=new THREE.Color(col);const hull=new THREE.Mesh(new THREE.OctahedronGeometry(1.5,0),makeMat(col,0.55,0.35,0.15));hull.scale.set(0.65,0.42,2.1);hull.position.z=-0.1;group.add(hull);const prow=new THREE.Mesh(new THREE.ConeGeometry(0.32,2.2,5),makeMat(col,0.9,0.2,0.08));prow.rotation.x=-Math.PI/2;prow.position.set(0,0,-2.4);group.add(prow);const swS=new THREE.Shape();swS.moveTo(0,0);swS.lineTo(4.2,-0.3);swS.lineTo(3.4,2.0);swS.lineTo(2.0,3.2);swS.lineTo(0.6,3.0);swS.lineTo(0,1.6);const swM=new THREE.MeshStandardMaterial({color:wG,emissive:wG.clone().multiplyScalar(0.35),side:THREE.DoubleSide,metalness:0.25,roughness:0.2,transparent:true,opacity:0.86});[-1,1].forEach(s=>{const sw=new THREE.Mesh(new THREE.ShapeGeometry(swS),swM.clone());sw.rotation.x=Math.PI/2;sw.scale.x=s;sw.position.set(s*0.6,-0.08,0.2);group.add(sw);});const halo=new THREE.Mesh(new THREE.TorusGeometry(2.1,0.1,10,36),new THREE.MeshBasicMaterial({color:0xfffacd,transparent:true,opacity:0.88}));halo.rotation.x=Math.PI*0.1;halo.position.set(0,0.7,-0.5);group.add(halo);seraphHaloRing=halo;const sOrb=new THREE.Mesh(new THREE.SphereGeometry(0.45,14,14),new THREE.MeshStandardMaterial({color:new THREE.Color(0xffffff),emissive:wG.clone(),emissiveIntensity:2.2,metalness:0,roughness:0,transparent:true,opacity:1.0}));sOrb.position.set(0,0.28,-0.7);group.add(sOrb);[-0.8,0.8].forEach(x=>{const tf=new THREE.Mesh(new THREE.BoxGeometry(0.08,1.1,0.9),makeMat(col,0.4,0.4,0.2));tf.position.set(x,0.25,1.55);tf.rotation.z=x>0?-0.15:0.15;group.add(tf);});}
  else if(id===9){const cr=new THREE.Color(col);const hull=new THREE.Mesh(new THREE.OctahedronGeometry(1.7,0),makeMat(col,0.5,0.7,0.15));hull.scale.set(0.58,0.33,2.3);hull.position.z=-0.2;group.add(hull);const prow=new THREE.Mesh(new THREE.ConeGeometry(0.28,2.8,4),makeMat(col,1.0,0.3,0.05));prow.rotation.x=-Math.PI/2;prow.position.set(0,0,-2.8);group.add(prow);const adS=new THREE.Shape();adS.moveTo(0,0);adS.lineTo(4.5,-0.6);adS.lineTo(4.0,0.5);adS.lineTo(2.8,1.4);adS.lineTo(1.0,1.8);adS.lineTo(0,0.9);const adM=new THREE.MeshStandardMaterial({color:cr,emissive:cr.clone().multiplyScalar(0.6),side:THREE.DoubleSide,metalness:0.5,roughness:0.15,transparent:true,opacity:0.82});[-1,1].forEach(s=>{const aw=new THREE.Mesh(new THREE.ShapeGeometry(adS),adM.clone());aw.rotation.x=Math.PI/2;aw.scale.x=s;aw.position.set(s*0.55,-0.05,0.1);group.add(aw);});const ehRing=new THREE.Mesh(new THREE.TorusGeometry(1.55,0.13,9,32),new THREE.MeshBasicMaterial({color:0xff4444,transparent:true,opacity:0.9}));ehRing.rotation.x=Math.PI*0.05;ehRing.position.set(0,0.1,-0.2);group.add(ehRing);const core=new THREE.Mesh(new THREE.SphereGeometry(0.55,14,14),new THREE.MeshStandardMaterial({color:0x110000,emissive:new THREE.Color(0xff1111),emissiveIntensity:1.8,metalness:0,roughness:0,transparent:true,opacity:1.0}));core.position.set(0,0.18,-0.5);group.add(core);}
  else if(id===10){const aC=new THREE.Color(col);const hull=new THREE.Mesh(new THREE.OctahedronGeometry(1.6,0),makeMat(col,0.65,0.15,0.05));hull.scale.set(0.55,0.38,2.4);hull.position.z=-0.15;group.add(hull);const prow=new THREE.Mesh(new THREE.ConeGeometry(0.24,3.2,6),makeMat(col,1.2,0.05,0.0));prow.rotation.x=-Math.PI/2;prow.position.set(0,0,-3.0);group.add(prow);const rfS=new THREE.Shape();rfS.moveTo(0,0);rfS.lineTo(4.0,-0.2);rfS.lineTo(3.6,1.0);rfS.lineTo(2.4,2.6);rfS.lineTo(1.2,3.0);rfS.lineTo(0.3,2.2);rfS.lineTo(0,1.0);const rfM=new THREE.MeshStandardMaterial({color:aC,emissive:aC.clone().multiplyScalar(0.5),side:THREE.DoubleSide,metalness:0.05,roughness:0.05,transparent:true,opacity:0.75});[-1,1].forEach(s=>{const rfw=new THREE.Mesh(new THREE.ShapeGeometry(rfS),rfM.clone());rfw.rotation.x=Math.PI/2;rfw.scale.x=s;rfw.position.set(s*0.52,-0.06,0.1);group.add(rfw);});const cr1=new THREE.Mesh(new THREE.TorusGeometry(1.85,0.08,8,32),new THREE.MeshBasicMaterial({color:0xaaaaff,transparent:true,opacity:0.8}));cr1.rotation.x=Math.PI*0.08;cr1.position.set(0,0.6,-0.3);group.add(cr1);const cr2=new THREE.Mesh(new THREE.TorusGeometry(2.3,0.05,6,28),new THREE.MeshBasicMaterial({color:0x6666cc,transparent:true,opacity:0.4}));cr2.rotation.x=Math.PI*0.08;cr2.rotation.z=Math.PI*0.15;cr2.position.set(0,0.6,-0.3);group.add(cr2);const cOrb=new THREE.Mesh(new THREE.SphereGeometry(0.52,14,14),new THREE.MeshStandardMaterial({color:new THREE.Color(0xffffff),emissive:new THREE.Color(0x8888ff),emissiveIntensity:2.5,metalness:0,roughness:0,transparent:true,opacity:1.0}));cOrb.position.set(0,0.25,-0.6);group.add(cOrb);const ghost=new THREE.Mesh(new THREE.OctahedronGeometry(1.6,0),new THREE.MeshStandardMaterial({color:new THREE.Color(0x4444cc),emissive:new THREE.Color(0x2222aa),emissiveIntensity:0.7,metalness:0,roughness:0.2,transparent:true,opacity:0.18}));ghost.scale.set(0.58,0.4,2.5);ghost.position.z=2.8;group.add(ghost);chronosGhostMesh=ghost;}
  else if(id===11){const hull=new THREE.Mesh(new THREE.ConeGeometry(0.72,5.2,5),makeMat(col,0.7,0.15,0.1));hull.rotation.x=-Math.PI/2;hull.position.z=-0.4;group.add(hull);const gwS=new THREE.Shape();gwS.moveTo(0,0);gwS.lineTo(5.0,0.4);gwS.lineTo(4.5,2.8);gwS.lineTo(2.5,4.5);gwS.lineTo(0.5,3.2);gwS.lineTo(0,1.8);const gwM=new THREE.MeshStandardMaterial({color:new THREE.Color(col),emissive:new THREE.Color(col).multiplyScalar(0.65),side:THREE.DoubleSide,metalness:0,roughness:0.2,transparent:true,opacity:0.58});[-1,1].forEach(s=>{const gw=new THREE.Mesh(new THREE.ShapeGeometry(gwS),gwM.clone());gw.rotation.x=Math.PI/2;gw.scale.x=s;gw.position.set(s*0.4,-0.1,0.4);group.add(gw);});const sw2S=new THREE.Shape();sw2S.moveTo(0,0);sw2S.lineTo(3.0,0.2);sw2S.lineTo(2.2,2.0);sw2S.lineTo(0.4,2.2);sw2S.lineTo(0,0.9);const sw2M=new THREE.MeshStandardMaterial({color:new THREE.Color(col),emissive:new THREE.Color(col).multiplyScalar(0.5),side:THREE.DoubleSide,metalness:0,roughness:0,transparent:true,opacity:0.42});[-1,1].forEach(s=>{const sw2=new THREE.Mesh(new THREE.ShapeGeometry(sw2S),sw2M.clone());sw2.rotation.x=Math.PI/2;sw2.scale.x=s;sw2.position.set(s*0.42,0.5,-0.3);group.add(sw2);});const vR=new THREE.Mesh(new THREE.TorusGeometry(0.65,0.06,8,20),new THREE.MeshBasicMaterial({color:new THREE.Color(col),transparent:true,opacity:0.88}));vR.rotation.x=Math.PI/2;vR.position.set(0,0.1,-0.6);group.add(vR);const vR2=new THREE.Mesh(new THREE.TorusGeometry(1.1,0.04,6,18),new THREE.MeshBasicMaterial({color:0xffffff,transparent:true,opacity:0.3}));vR2.rotation.x=Math.PI/2;vR2.position.set(0,0.1,-0.6);group.add(vR2);const cOrb=new THREE.Mesh(new THREE.SphereGeometry(0.38,10,10),new THREE.MeshStandardMaterial({color:0x110022,emissive:new THREE.Color(col),emissiveIntensity:2.0,metalness:0,roughness:0,transparent:true,opacity:1.0}));cOrb.position.set(0,0.15,-0.5);group.add(cOrb);const orbColors=[col,"#ffffff","#ee88ff"],orbs=[];orbColors.forEach(oc=>{const o=new THREE.Mesh(new THREE.SphereGeometry(0.19,6,6),new THREE.MeshStandardMaterial({color:new THREE.Color(oc),emissive:new THREE.Color(oc),emissiveIntensity:1.8,metalness:0,roughness:0}));group.add(o);orbs.push(o);});group.userData.wraithOrbs=orbs;[-0.6,0.6].forEach(x=>{const st=new THREE.Mesh(new THREE.BoxGeometry(0.07,0.85,0.8),makeMat(col).clone());st.position.set(x,0.1,1.55);group.add(st);});}
  else if(id===12){const hM=makeMat(col,0.45,0.7,0.2);const hull=new THREE.Mesh(new THREE.BoxGeometry(2.3,0.85,4.0),hM);group.add(hull);[-1.85,1.85].forEach(x=>{const p=new THREE.Mesh(new THREE.BoxGeometry(0.75,0.62,2.6),hM.clone());p.position.set(x,0.18,-0.2);group.add(p);});const nose=new THREE.Mesh(new THREE.ConeGeometry(1.18,1.9,4),hM.clone());nose.rotation.x=-Math.PI/2;nose.position.set(0,0,-2.9);group.add(nose);group.add(new THREE.Mesh(new THREE.BoxGeometry(2.3,0.85,0.4),hM.clone())).position.set(0,0,-2.0);const dome=new THREE.Mesh(new THREE.SphereGeometry(0.78,8,6,0,Math.PI*2,0,Math.PI/2),makeMat(col,0.9,0.1,0.05));dome.position.set(0,0.5,-0.5);group.add(dome);[-0.8,0,0.8].forEach(z=>{const rib=new THREE.Mesh(new THREE.BoxGeometry(2.8,0.15,0.15),hM.clone());rib.position.set(0,0.48,z);group.add(rib);});[-0.85,0.85].forEach(x=>{const pod=new THREE.Mesh(new THREE.CylinderGeometry(0.3,0.38,1.3,7),hM.clone());pod.rotation.x=Math.PI/2;pod.position.set(x,-0.2,2.1);group.add(pod);const glow=new THREE.Mesh(new THREE.CircleGeometry(0.28,10),new THREE.MeshBasicMaterial({color:new THREE.Color(col),transparent:true,opacity:0.8,side:THREE.DoubleSide}));glow.rotation.y=Math.PI;glow.position.set(x,-0.2,2.75);group.add(glow);});}
  else if(id===13){const oC=new THREE.Color(col);const hull=new THREE.Mesh(new THREE.OctahedronGeometry(1.55,0),makeMat(col,0.6,0.5,0.08));hull.scale.set(0.68,0.45,2.2);hull.position.z=-0.15;group.add(hull);const spear=new THREE.Mesh(new THREE.ConeGeometry(0.28,3.2,5),makeMat(col,1.3,0.3,0.0));spear.rotation.x=-Math.PI/2;spear.position.set(0,0,-2.9);group.add(spear);const owS=new THREE.Shape();owS.moveTo(0,0);owS.lineTo(5.2,-0.4);owS.lineTo(4.8,0.9);owS.lineTo(3.2,2.6);owS.lineTo(1.0,3.0);owS.lineTo(0,1.8);const owM=new THREE.MeshStandardMaterial({color:oC,emissive:oC.clone().multiplyScalar(0.5),side:THREE.DoubleSide,metalness:0.4,roughness:0.1,transparent:true,opacity:0.85});[-1,1].forEach(s=>{const ow=new THREE.Mesh(new THREE.ShapeGeometry(owS),owM.clone());ow.rotation.x=Math.PI/2;ow.scale.x=s;ow.position.set(s*0.65,-0.1,0.2);group.add(ow);});const wsS=new THREE.Shape();wsS.moveTo(0,0);wsS.lineTo(3.2,0.1);wsS.lineTo(2.4,1.6);wsS.lineTo(0.5,1.8);wsS.lineTo(0,0.8);const wsM=new THREE.MeshStandardMaterial({color:0xffffff,emissive:oC,emissiveIntensity:0.6,side:THREE.DoubleSide,transparent:true,opacity:0.45});[-1,1].forEach(s=>{const ws=new THREE.Mesh(new THREE.ShapeGeometry(wsS),wsM.clone());ws.rotation.x=Math.PI/2;ws.scale.x=s;ws.position.set(s*0.66,0.4,-0.2);group.add(ws);});[{r:2.0,t:0.10,p:[0,0.3,-0.6],rx:Math.PI*0.06,rz:0},{r:2.5,t:0.06,p:[0,0.3,-0.2],rx:Math.PI*0.04,rz:Math.PI*0.12},{r:1.6,t:0.07,p:[0,0.3,-1.0],rx:Math.PI*0.10,rz:-Math.PI*0.08}].forEach(({r,t,p,rx,rz})=>{const rm=new THREE.Mesh(new THREE.TorusGeometry(r,t,8,32),new THREE.MeshBasicMaterial({color:new THREE.Color(col),transparent:true,opacity:0.85}));rm.rotation.x=rx;rm.rotation.z=rz;rm.position.set(...p);group.add(rm);});const oOrb=new THREE.Mesh(new THREE.SphereGeometry(0.55,14,14),new THREE.MeshStandardMaterial({color:0xffffff,emissive:oC.clone(),emissiveIntensity:2.8,metalness:0,roughness:0,transparent:true,opacity:1.0}));oOrb.position.set(0,0.2,-0.5);group.add(oOrb);const hot=new THREE.Mesh(new THREE.SphereGeometry(0.22,8,8),new THREE.MeshBasicMaterial({color:0xffffff,transparent:true,opacity:1.0}));hot.position.set(0,0.2,-0.5);group.add(hot);[-0.75,0.75].forEach(x=>{const tf=new THREE.Mesh(new THREE.BoxGeometry(0.07,1.1,0.9),makeMat(col,0.5,0.5,0.1));tf.position.set(x,0.22,1.6);tf.rotation.z=x>0?-0.15:0.15;group.add(tf);});}
  else if(id===14){
    // Stellar Sovereign — chromatic pass ship
    const sC=new THREE.Color("#cc44ff");const sC2=new THREE.Color("#ff44bb");const sC3=new THREE.Color("#ffcc00");
    const hull=new THREE.Mesh(new THREE.OctahedronGeometry(1.5,0),makeMat("#cc44ff",0.8,0.1,0.05));
    hull.scale.set(0.6,0.4,2.3);hull.position.z=-0.2;group.add(hull);
    const prow=new THREE.Mesh(new THREE.ConeGeometry(0.26,3.0,6),makeMat("#ffffff",1.5,0.0,0.0));
    prow.rotation.x=-Math.PI/2;prow.position.set(0,0,-2.8);group.add(prow);
    // Three layered wing shapes in different chromatic colors
    const wColors=["#ff44bb","#ffcc00","#00ffcc"];
    const wDefs=[
      {pts:[[0,0],[5.0,-0.3],[4.5,1.2],[2.8,2.8],[0.8,3.0],[0,1.6]],pos:[0.5,-0.05,0.2]},
      {pts:[[0,0],[3.5,0.1],[2.8,1.8],[0.6,2.0],[0,1.0]],pos:[0.52,0.4,-0.3]},
      {pts:[[0,0],[2.2,0.0],[1.6,1.0],[0.3,1.2],[0,0.6]],pos:[0.54,0.8,-0.8]},
    ];
    wDefs.forEach(({pts,pos},wi)=>{
      const sh=new THREE.Shape();sh.moveTo(pts[0][0],pts[0][1]);pts.slice(1).forEach(p=>sh.lineTo(p[0],p[1]));
      const wm=new THREE.MeshStandardMaterial({color:new THREE.Color(wColors[wi]),emissive:new THREE.Color(wColors[wi]).multiplyScalar(0.6),side:THREE.DoubleSide,transparent:true,opacity:0.72-wi*0.1,metalness:0,roughness:0.1});
      [-1,1].forEach(s=>{const w=new THREE.Mesh(new THREE.ShapeGeometry(sh),wm.clone());w.rotation.x=Math.PI/2;w.scale.x=s;w.position.set(s*pos[0],pos[1],pos[2]);group.add(w);});
    });
    // Triple rings
    [{r:2.2,t:0.1,c:"#cc44ff"},{r:2.8,t:0.06,c:"#ff44bb"},{r:1.6,t:0.07,c:"#ffcc00"}].forEach(({r,t,c},ri)=>{
      const ring=new THREE.Mesh(new THREE.TorusGeometry(r,t,8,32),new THREE.MeshBasicMaterial({color:new THREE.Color(c),transparent:true,opacity:0.88-ri*0.1}));
      ring.rotation.x=Math.PI*(0.06+ri*0.04);ring.rotation.z=Math.PI*(ri*0.08);ring.position.set(0,0.3,-0.5);group.add(ring);
    });
    // Core orb — white hot center
    const sovOrb=new THREE.Mesh(new THREE.SphereGeometry(0.6,14,14),new THREE.MeshStandardMaterial({color:0xffffff,emissive:new THREE.Color("#cc44ff"),emissiveIntensity:3.0,metalness:0,roughness:0,transparent:true,opacity:1.0}));
    sovOrb.position.set(0,0.2,-0.5);group.add(sovOrb);
    const hotCore=new THREE.Mesh(new THREE.SphereGeometry(0.25,8,8),new THREE.MeshBasicMaterial({color:0xffffff,transparent:true,opacity:1.0}));
    hotCore.position.set(0,0.2,-0.5);group.add(hotCore);
    [-0.8,0.8].forEach(x=>{const tf=new THREE.Mesh(new THREE.BoxGeometry(0.07,1.2,1.0),makeMat("#cc44ff",0.7,0.1,0.1));tf.position.set(x,0.22,1.65);tf.rotation.z=x>0?-0.12:0.12;group.add(tf);});
  }
  // Engine nozzle
  const nozzle=new THREE.Mesh(new THREE.CylinderGeometry(0.3,0.5,0.4,7),new THREE.MeshStandardMaterial({color:0x222244,emissive:new THREE.Color(col).multiplyScalar(0.5),metalness:0.8,roughness:0.2}));
  nozzle.rotation.x=Math.PI/2;nozzle.position.set(0,0,1.5);group.add(nozzle);
  const glow=new THREE.Mesh(new THREE.CircleGeometry(0.55,12),new THREE.MeshBasicMaterial({color:new THREE.Color(col),transparent:true,opacity:0.7,side:THREE.DoubleSide}));
  glow.rotation.y=Math.PI;glow.position.set(0,0,1.75);group.add(glow);
  group.scale.set(0.85,0.85,0.85);
  return group;
}

// ASTEROIDS
const ASTEROID_GEO=(()=>{const g=new THREE.IcosahedronGeometry(1.1,1);const pos=g.attributes.position;for(let i=0;i<pos.count;i++){const s=0.7+Math.random()*0.5;pos.setXYZ(i,pos.getX(i)*s,pos.getY(i)*s,pos.getZ(i)*s);}g.computeVertexNormals();return g;})();
const ASTEROID_MATS=[new THREE.MeshStandardMaterial({color:0x776655,roughness:0.95,metalness:0.05}),new THREE.MeshStandardMaterial({color:0x665544,roughness:0.9,metalness:0.1}),new THREE.MeshStandardMaterial({color:0x888877,roughness:0.92,metalness:0.08})];
const G2_ASTEROID_MATS=[new THREE.MeshStandardMaterial({color:0x442255,roughness:0.82,metalness:0.25,emissive:new THREE.Color(0x221133),emissiveIntensity:0.5}),new THREE.MeshStandardMaterial({color:0x553377,roughness:0.78,metalness:0.3,emissive:new THREE.Color(0x2a1155),emissiveIntensity:0.4}),new THREE.MeshStandardMaterial({color:0x331144,roughness:0.88,metalness:0.18,emissive:new THREE.Color(0x1a0033),emissiveIntensity:0.6})];
function createAsteroidMesh(){const mats=isGalaxy2()?G2_ASTEROID_MATS:ASTEROID_MATS;const mat=mats[Math.floor(Math.random()*mats.length)].clone();const mesh=new THREE.Mesh(ASTEROID_GEO,mat);const s=0.8+Math.random()*0.6;mesh.scale.set(s,s*(0.8+Math.random()*0.4),s*(0.8+Math.random()*0.4));mesh.rotation.set(Math.random()*Math.PI*2,Math.random()*Math.PI*2,Math.random()*Math.PI*2);return mesh;}
function createShieldMesh(){const mesh=new THREE.Mesh(new THREE.SphereGeometry(2.2,18,14),new THREE.MeshBasicMaterial({color:0x00ffff,wireframe:true,transparent:true,opacity:0.35}));mesh.visible=false;return mesh;}

// GHOST DECOY
function createGhostDecoy(fromLane){
  if(ghostDecoy){scene.remove(ghostDecoy);ghostDecoy=null;}
  const g=new THREE.Group();const col="#ff77ff";const gM=new THREE.MeshStandardMaterial({color:new THREE.Color(col),emissive:new THREE.Color(col).multiplyScalar(0.8),transparent:true,opacity:0.38,metalness:0.1,roughness:0.5});
  const hull=new THREE.Mesh(new THREE.ConeGeometry(1.1,4.5,3),gM.clone());hull.rotation.x=-Math.PI/2;hull.position.z=-0.2;g.add(hull);
  const orb=new THREE.Mesh(new THREE.SphereGeometry(0.42,8,8),gM.clone());orb.position.set(0,0.22,-0.6);g.add(orb);
  g.scale.set(0.85,0.85,0.85);g.position.set(LANES_3D[fromLane],0,0);scene.add(g);
  ghostDecoy=g;ghostDecoyLane=fromLane;ghostDecoyFrames=300;
}
function updateGhostDecoy(){
  if(!ghostDecoy)return;ghostDecoyFrames--;
  const pulse=0.2+Math.sin(Date.now()*0.006)*0.18;
  ghostDecoy.traverse(c=>{if(c.material&&c.material.opacity!==undefined)c.material.opacity=Math.max(0,pulse*(ghostDecoyFrames/300));});
  ghostDecoy.rotation.y+=0.015;
  if(ghostDecoyFrames<=0){scene.remove(ghostDecoy);ghostDecoy=null;ghostDecoyLane=-1;}
}

// PARTICLES
function spawnVoidBurst(x,y,z){const cols=[0xff44ff,0xaa00ff,0xffffff,0xff99ff,0x6600ff];for(let i=0;i<28;i++){const mesh=new THREE.Mesh(new THREE.SphereGeometry(0.06+Math.random()*0.12,4,4),new THREE.MeshBasicMaterial({color:cols[Math.floor(Math.random()*cols.length)],transparent:true,opacity:0.95,blending:THREE.AdditiveBlending}));mesh.position.set(x+(Math.random()-.5)*1.5,y+(Math.random()-.5)*1.5,z+(Math.random()-.5)*1.5);const sp=0.2+Math.random()*0.18,angle=Math.random()*Math.PI*2;voidParticles.push({mesh,vx:Math.cos(angle)*sp*(Math.random()-.5)*2,vy:(Math.random()-.5)*sp,vz:Math.sin(angle)*sp*(Math.random()-.5)*2,life:30+Math.floor(Math.random()*20),maxLife:50});scene.add(mesh);}}
function updateVoidParticles(){for(let i=voidParticles.length-1;i>=0;i--){const p=voidParticles[i];p.mesh.position.x+=p.vx;p.mesh.position.y+=p.vy;p.mesh.position.z+=p.vz;p.vx*=0.92;p.vy*=0.92;p.vz*=0.92;p.life--;p.mesh.material.opacity=(p.life/p.maxLife)*0.9;p.mesh.scale.setScalar(Math.max(0.05,p.life/p.maxLife));if(p.life<=0){scene.remove(p.mesh);voidParticles.splice(i,1);}}}
function spawnSoulBurst(){const cols=[0xfffacd,0xfff080,0xffd700,0xffffff,0xffe4a0,0xffcc44];const sx=ship.x;for(let i=0;i<48;i++){const mesh=new THREE.Mesh(new THREE.SphereGeometry(0.1+Math.random()*0.18,4,4),new THREE.MeshBasicMaterial({color:cols[Math.floor(Math.random()*cols.length)],transparent:true,opacity:1.0,blending:THREE.AdditiveBlending}));mesh.position.set(sx+(Math.random()-.5)*2,(Math.random()-.5)*2,(Math.random()-.5)*4);const speed=0.28+Math.random()*0.24,angle=Math.random()*Math.PI*2,elev=(Math.random()-.5)*Math.PI;soulParticles.push({mesh,vx:Math.cos(angle)*Math.cos(elev)*speed,vy:Math.sin(elev)*speed*0.6,vz:Math.sin(angle)*Math.cos(elev)*speed-0.08,life:50+Math.floor(Math.random()*20),maxLife:70});scene.add(mesh);}seraphSoulBurstFlashFrames=45;soulLight.position.set(sx,1,0);soulLight.intensity=22;}
function updateSoulParticles(){for(let i=soulParticles.length-1;i>=0;i--){const p=soulParticles[i];p.mesh.position.x+=p.vx;p.mesh.position.y+=p.vy;p.mesh.position.z+=p.vz;p.vx*=0.93;p.vy*=0.93;p.vz*=0.93;p.life--;p.mesh.material.opacity=Math.pow(p.life/p.maxLife,0.7)*0.95;p.mesh.scale.setScalar(Math.max(0.04,p.life/p.maxLife*1.3));if(p.life<=0){scene.remove(p.mesh);soulParticles.splice(i,1);}}if(seraphSoulBurstFlashFrames>0)seraphSoulBurstFlashFrames--;if(soulLight.intensity>0)soulLight.intensity=Math.max(0,soulLight.intensity*0.88);}
function spawnEngineParticle(x,col){const mesh=new THREE.Mesh(new THREE.SphereGeometry(0.08,4,4),new THREE.MeshBasicMaterial({color:new THREE.Color(col),transparent:true,opacity:0.9}));mesh.position.set(x+(Math.random()-.5)*0.5,(Math.random()-.5)*0.4,1.8+(Math.random()*.5));scene.add(mesh);engineExhaust.push({mesh,life:18,maxLife:18,vz:(0.1+Math.random()*0.15)});}
function updateEngineExhaust(){for(let i=engineExhaust.length-1;i>=0;i--){const p=engineExhaust[i];p.mesh.position.z+=p.vz;p.mesh.position.y-=0.02;p.life--;p.mesh.material.opacity=(p.life/p.maxLife)*0.7;p.mesh.scale.setScalar(p.life/p.maxLife);if(p.life<=0){scene.remove(p.mesh);engineExhaust.splice(i,1);}}}
function spawnParticles3D(x,y,z,n){for(let i=0;i<n;i++){const col=isGalaxy2()?(Math.random()>.5?0xdd44ff:0x9900ff):(Math.random()>.5?0xff6600:0xffaa00);const mesh=new THREE.Mesh(new THREE.SphereGeometry(0.14,3,3),new THREE.MeshBasicMaterial({color:col,transparent:true,opacity:0.95}));mesh.position.set(x+(Math.random()-.5)*2,y+(Math.random()-.5)*2,z+(Math.random()-.5)*2);const sp=0.12;particles3D.push({mesh,vx:(Math.random()-.5)*sp*2,vy:(Math.random()-.5)*sp*2,vz:(Math.random()-.5)*sp*2,life:40,maxLife:40});scene.add(mesh);}}
function updateParticles3D(){for(let i=particles3D.length-1;i>=0;i--){const p=particles3D[i];p.mesh.position.x+=p.vx;p.mesh.position.y+=p.vy;p.mesh.position.z+=p.vz;p.life--;p.mesh.material.opacity=(p.life/p.maxLife)*0.9;p.mesh.scale.setScalar(Math.max(0.1,p.life/p.maxLife));if(p.life<=0){scene.remove(p.mesh);particles3D.splice(i,1);}}}

// GEMS
function spawnGem(laneIndex){const col=isGalaxy2()?0xff88ff:0x00ffd0;const eCol=isGalaxy2()?new THREE.Color(0xff88ff):new THREE.Color(0x00ffd0);const mesh=new THREE.Mesh(new THREE.OctahedronGeometry(0.4,0),new THREE.MeshStandardMaterial({color:col,emissive:eCol,emissiveIntensity:1.4,metalness:0.3,roughness:0.1,transparent:true,opacity:0.95}));mesh.position.set(LANES_3D[laneIndex],0.4,SPAWN_Z);scene.add(mesh);gemObjects.push({mesh,lane:laneIndex,z:SPAWN_Z});}
function clearGems(){gemObjects.forEach(g=>scene.remove(g.mesh));gemObjects=[];}
function updateGems(spd3D){for(let i=gemObjects.length-1;i>=0;i--){const g=gemObjects[i];g.z+=spd3D;g.mesh.position.z=g.z;g.mesh.rotation.y+=0.06;if(isHarbinger()){const tx=LANES_3D[ship.lane];g.mesh.position.x+=(tx-g.mesh.position.x)*0.04;}if(g.z>PASS_Z-2&&g.z<PASS_Z+2){const cr=isHarbinger()?4.5:1.8;if(Math.abs(g.mesh.position.x-ship.x)<cr){gems++;saveProgress();updateAllRankUi();spawnParticles3D(g.mesh.position.x,g.mesh.position.y,g.mesh.position.z,6);showToast("💎 +1 Gem!",900);scene.remove(g.mesh);gemObjects.splice(i,1);continue;}}if(g.z>PASS_Z+3){scene.remove(g.mesh);gemObjects.splice(i,1);}}}

// LASER
let laserFrames3D=0,laserLane3D=0;
function updateLaser3D(){if(laserBeam){if(laserFrames3D>0){laserBeam.material.opacity=laserFrames3D/20;laserFrames3D--;}else{scene.remove(laserBeam);laserBeam=null;}}}
function showLaserBeam(laneX){if(laserBeam)scene.remove(laserBeam);const mesh=new THREE.Mesh(new THREE.CylinderGeometry(0.08,0.08,140,6),new THREE.MeshBasicMaterial({color:0xff4400,transparent:true,opacity:1,blending:THREE.AdditiveBlending}));mesh.rotation.x=Math.PI/2;mesh.position.set(laneX,0,-60);scene.add(mesh);laserBeam=mesh;laserFrames3D=20;}

// SCREEN FLASHES
function doScreenFlash(g,dur=300){const el=document.getElementById("teleportFlash");el.style.background=g;el.style.display="block";el.classList.remove("teleport-anim");void el.offsetWidth;el.classList.add("teleport-anim");setTimeout(()=>{el.style.display="none";el.classList.remove("teleport-anim");},dur);}
function doTeleportScreenFlash(){doScreenFlash("radial-gradient(ellipse at center,rgba(255,80,255,0.7) 0%,rgba(120,0,200,0.4) 50%,transparent 100%)");}
function doSoulBurstScreenFlash(){doScreenFlash("radial-gradient(ellipse at center,rgba(255,250,150,0.75) 0%,rgba(255,200,50,0.45) 45%,transparent 100%)",500);}
function doSingularityFlash(){doScreenFlash("radial-gradient(ellipse at center,rgba(255,60,60,0.55) 0%,rgba(180,0,0,0.25) 50%,transparent 100%)",400);}
function doChronosFlash(e){doScreenFlash(e?"radial-gradient(ellipse at center,rgba(180,180,255,0.6) 0%,rgba(80,80,200,0.3) 50%,transparent 100%)":"radial-gradient(ellipse at center,rgba(255,255,255,0.45) 0%,rgba(150,150,255,0.2) 50%,transparent 100%)",350);}
function doDarkPulseFlash(){doScreenFlash("radial-gradient(ellipse at center,rgba(200,50,255,0.65) 0%,rgba(100,0,180,0.3) 50%,transparent 100%)",400);}
function doOmegaFlash(){doScreenFlash("radial-gradient(ellipse at center,rgba(255,150,0,0.6) 0%,rgba(200,80,0,0.3) 50%,transparent 100%)",450);}

// GAME STATE
let score,highScore,gameOver,baseSpeed,boostKey,difficultyTimer;
let xp,xpMilestoneNext,xpPassiveTimer,shieldActive,dodgedCount;
let legendRevives=0,frozenFrames=0,godlyCooldown=0;
let boostedShips=[],gems=0,chosenCosmetic=0;
let startShieldActive=false,startShieldFrames=0,activeRunBoosts=[];
let laserShots=0,laserFrames=0,laserLane=0;
let playerName="Pilot",questRunScore=0;
let animationFrameId=null,idleAnimId=null,mobileControls=false;
let gemSpawnTimer=0,darkPulseCounter=0;
const ship={lane:1,x:0,targetX:0,moveSpeed:.11,tilt:0};

// HELPERS
function isSeraph(){return chosenCosmetic===8;}
function isPhantomatic(){return chosenCosmetic===7;}
function isHarbinger(){return chosenCosmetic===9;}
function isAeon(){return chosenCosmetic===10;}
function isWraith(){return chosenCosmetic===11;}
function isColossus(){return chosenCosmetic===12;}
function isOmega(){return chosenCosmetic===13;}
function isSovereign(){return chosenCosmetic===14;}
function isGalaxy2(){return chosenCosmetic>=11&&chosenCosmetic<=13;}
let chromaticSurgeCounter=0;
function hasRunBoost(id){return activeRunBoosts.includes(id);}
function getPowerActive(name){return COSMETICS[chosenCosmetic].powerName===name;}
function isBoosted(id){return boostedShips.includes(id);}
function hasAnnihilateAbility(){return getPowerActive("Time Annihilate")||isOmega();}
function getBoostMultiplier(){if(chosenCosmetic===1&&isBoosted(1))return 4;if(getPowerActive("Double Boost"))return 2;return 1;}
function getAsteroidSpeedMult(){let m=1;if(chosenCosmetic===4&&isBoosted(4))m*=.6;else if(getPowerActive("Slow Asteroids"))m*=.8;if(hasRunBoost("null_field"))m*=.65;return m;}
function getMilestoneXpBonus(){if(chosenCosmetic===3&&isBoosted(3))return 2;if(getPowerActive("Milestone XP+"))return 1.5;return 1;}
function getShieldEveryDodged(){if(chosenCosmetic===2&&isBoosted(2))return 2;if(getPowerActive("Shield on Dodge"))return 5;if(isColossus())return isBoosted(12)?10:20;return null;}
function getLegendRevives(){if(chosenCosmetic===5&&isBoosted(5))return 2;if(getPowerActive("Revive"))return 1;return 0;}
function getGodlyCooldownFrames(){if(isOmega()&&isBoosted(13))return 600;if(isOmega())return 1200;if(chosenCosmetic===6&&isBoosted(6))return 900;return 1800;}
function getSoulBurstEvery(){if(!isSeraph())return 999999;return isBoosted(8)?10:20;}
function getDarkPulseEvery(){if(!isWraith())return 999999;return isBoosted(11)?8:15;}
function getBoostedDesc(id){return["Start invulnerable (shield) for 10s","Boost is 4×","Shield every 2 dodges","+100% XP for milestones","Asteroid speed −40%","Two revives per game","Half cooldown: F every 15s","Ghost decoy on teleport — absorbs 1 hit","Soul Burst every 10 dodges + double XP burst","Singularity pulse every 8s + gems auto-collected from any lane","Chronos phase every 5s (3s invincible) + ghost trail always visible","Dark Pulse every 8 dodges + XP burst per asteroid cleared","Adjacent lanes −40% speed + auto-shield every 10 dodges","Cosmic Reckoning every 10s + earn up to 8 💎"][id]||"";}
function flashScreen(color){const f=document.getElementById("reviveFlash");f.style.background=color;f.style.display="block";setTimeout(()=>f.style.display="none",300);}

// POWER TRIGGERS
function triggerSoulBurst(){const count=asteroids.length;asteroids.forEach(a=>{spawnParticles3D(a.x,0,a.z,6);if(a.mesh)scene.remove(a.mesh);});asteroids=[];const xpGain=isBoosted(8)?count*30:count*15;if(xpGain>0)addXp(xpGain);spawnSoulBurst();doSoulBurstScreenFlash();showToast(isBoosted(8)?`✨ SOUL BURST! +${xpGain} XP (BOOSTED)`:`✨ SOUL BURST! +${xpGain} XP`,2200);}
function triggerSingularityPulse(){const tx=LANES_3D[ship.lane];gemObjects.forEach(g=>{g.mesh.position.x=tx;g.lane=ship.lane;});doSingularityFlash();singLight.position.set(ship.x,0,0);singLight.intensity=12;showToast("🌀 Singularity Pulse — all gems drawn in!",1800);}
function triggerDarkPulse(){const px=ship.x;let cleared=0;for(let i=asteroids.length-1;i>=0;i--){if(Math.abs(asteroids[i].x-px)<1){spawnParticles3D(asteroids[i].x,0,asteroids[i].z,5);scene.remove(asteroids[i].mesh);asteroids.splice(i,1);cleared++;}}shieldActive=true;const bXp=isBoosted(11)?cleared*12:0;if(bXp>0)addXp(bXp);doDarkPulseFlash();voidLight.color.set(0xdd44ff);voidLight.position.set(ship.x,0,0);voidLight.intensity=12;showToast(`💜 Dark Pulse! ${cleared} cleared + Shield${bXp?` +${bXp}XP`:""}`,1600);}
function triggerCosmicReckoning(){const count=asteroids.length;asteroids.forEach(a=>{spawnParticles3D(a.x,0,a.z,5);scene.remove(a.mesh);});asteroids=[];frozenFrames=300;godlyCooldown=getGodlyCooldownFrames();const maxG=isBoosted(13)?8:5;const gemGain=Math.min(maxG,Math.max(1,Math.floor(count/2)));if(gemGain>0){gems+=gemGain;saveProgress();updateAllRankUi();}doOmegaFlash();singLight.color.set(0xff8800);singLight.position.set(ship.x,0,0);singLight.intensity=18;showToast(`🔥 COSMIC RECKONING! ${count} cleared +${gemGain}💎`,2200);}
function doPhantomTeleport(targetLane){if(isTeleporting||gameOver)return;const fromLane=ship.lane;if(fromLane===targetLane)return;const fromX=LANES_3D[fromLane],toX=LANES_3D[targetLane];isTeleporting=true;teleportFrames=TELEPORT_DURATION;spawnVoidBurst(fromX,0,0);if(isBoosted(7))createGhostDecoy(fromLane);if(shipGroup)shipGroup.visible=false;doTeleportScreenFlash();voidLight.position.set(fromX,0,0);voidLight.intensity=18;setTimeout(()=>{ship.lane=targetLane;ship.x=toX;ship.targetX=toX;if(shipGroup){shipGroup.position.x=toX;shipGroup.visible=true;}spawnVoidBurst(toX,0,0);doTeleportScreenFlash();voidLight.position.set(toX,0,0);voidLight.intensity=14;isTeleporting=false;},TELEPORT_DURATION*16);}

// PERSISTENCE
function saveProgress(){localStorage.setItem("sd2_xp",xp);localStorage.setItem("sd2_cosmetic",chosenCosmetic);localStorage.setItem("sd2_boosted",JSON.stringify(boostedShips));localStorage.setItem("sd2_gems",gems);localStorage.setItem("sd2_name",playerName);localStorage.setItem("sd2_mobile",mobileControls?1:0);}
function loadProgress(){xp=parseInt(localStorage.getItem("sd2_xp")||"0")||0;chosenCosmetic=parseInt(localStorage.getItem("sd2_cosmetic")||"0")||0;boostedShips=JSON.parse(localStorage.getItem("sd2_boosted")||"[]");gems=parseInt(localStorage.getItem("sd2_gems")||"0")||0;playerName=localStorage.getItem("sd2_name")||"Pilot";mobileControls=localStorage.getItem("sd2_mobile")==="1";xpMilestoneNext=XP_MILESTONE_STEP;document.getElementById("playerNameInput").value=playerName;}

// UI
function updateGemsUi(){document.querySelectorAll(".gemCount").forEach(el=>el.innerText=gems);}
function getXpRank(v){for(let i=XP_RANKS.length-1;i>=0;i--)if(v>=XP_RANKS[i].xp)return XP_RANKS[i];return XP_RANKS[0];}
function nextXpRank(){const idx=XP_RANKS.findIndex(r=>r.xp===getXpRank(xp).xp);return XP_RANKS[idx+1]||null;}
function getXpProgress(){const next=nextXpRank();if(!next)return 100;const cur=getXpRank(xp);return Math.min(100,Math.max(0,(xp-cur.xp)/(next.xp-cur.xp)*100));}
function getXpProgressText(){const next=nextXpRank();return next?`${xp} / ${next.xp} XP`:`${xp} / MAX`;}
function updateAllRankUi(){
  const rank=getXpRank(xp),pct=getXpProgress(),txt=getXpProgressText();
  updateGemsUi();
  const hr=document.getElementById("homeRank");hr.innerText=rank.name.toUpperCase();hr.style.color=rank.color;
  document.getElementById("homeXpLabel").innerText="XP: "+xp;
  document.getElementById("homeRankFill").style.width=pct+"%";document.getElementById("homeRankFill").style.background=`linear-gradient(90deg,${rank.color},#ffff00)`;
  document.getElementById("homeRankText").innerText=txt;
  const rd=document.getElementById("rankDisplay");rd.innerText=rank.name.toUpperCase();rd.style.color=rank.color;
  document.getElementById("xpDisplay").innerText="XP: "+xp;
  document.getElementById("gameRankFill").style.width=pct+"%";document.getElementById("gameRankFill").style.background=`linear-gradient(90deg,${rank.color},#ffff00)`;
  document.getElementById("gameRankText").innerText=txt;
  document.getElementById("galaxyBanner").style.display=xp>=20000?"block":"none";
  if(document.getElementById("homeMain").style.display!=="none")updateHomeShipCard();
}
function addXp(amount){const mult=hasRunBoost("xp_surge")?3:1;const oldRank=getXpRank(xp);xp+=amount*mult;const newRank=getXpRank(xp);if(newRank.name!==oldRank.name){gems++;updateGemsUi();saveProgress();showToast("+1 💎 for reaching "+newRank.name+"!");triggerRankUpAnimation();}saveProgress();updateAllRankUi();}
function triggerRankUpAnimation(){const el=document.getElementById("rankUpAnimation");el.style.display="block";el.offsetHeight;setTimeout(()=>el.style.display="none",1100);}
function showToast(msg,duration=2200){const t=document.getElementById("toast");t.innerText=msg;t.style.display="block";clearTimeout(t._tid);t._tid=setTimeout(()=>t.style.display="none",duration);}

// BOOSTER
function runBoosterLogic(){
  if(gems<1){showToast("Not enough 💎 Gems!");return undefined;}
  const avail=COSMETICS.map(c=>c.id).filter(id=>!isBoosted(id));
  if(!avail.length){showToast("All ships already BOOSTED!");return undefined;}
  const r=avail[Math.floor(Math.random()*avail.length)];
  boostedShips.push(r);gems--;saveProgress();updateAllRankUi();
  showToast(`BOOSTED: ${COSMETICS[r].name} — ${getBoostedDesc(r)}`,4000);
  return r;
}

// RUN BOOSTS
function toggleRunBoost(id){const def=RUN_BOOSTS.find(b=>b.id===id);if(!def)return;if(hasRunBoost(id)){activeRunBoosts=activeRunBoosts.filter(b=>b!==id);gems+=def.cost;saveProgress();updateAllRankUi();}else{if(gems<def.cost){showToast("Not enough 💎 Gems!");return;}gems-=def.cost;activeRunBoosts.push(id);saveProgress();updateAllRankUi();}renderBoostsTab();}
function renderBoostsTab(){
  const grid=document.getElementById("boostsGrid");grid.innerHTML="";
  RUN_BOOSTS.forEach(b=>{const eq=hasRunBoost(b.id),afford=gems>=b.cost;const d=document.createElement("div");d.className="boostCard"+(eq?" equipped":"");d.innerHTML=`<div class="bn">${b.name}</div><div class="bd">${b.desc}</div><div class="bc">${eq?"✅ Equipped":"Cost: "+b.cost+" 💎"}</div><button class="${eq?'unequip':''}" ${!eq&&!afford?"disabled":""} onclick="toggleRunBoost('${b.id}')">${eq?"Unequip":"Equip"}</button>`;grid.appendChild(d);});
  const el=document.getElementById("activeBoostsList");
  if(!activeRunBoosts.length){el.innerText="No run boosts equipped.";return;}
  el.innerText="Active: "+activeRunBoosts.map(id=>RUN_BOOSTS.find(b=>b.id===id).name).join(" | ");
}

// QUESTS
function loadQuestState(){try{return JSON.parse(localStorage.getItem("sd2_qstate")||"null")}catch(e){return null}}
function saveQuestState(s){localStorage.setItem("sd2_qstate",JSON.stringify(s));}
function seededRng(seed){let s=seed;return()=>{s=(s*9301+49297)%233280;return s/233280;};}
function generateBatch(batchIndex){const tier=QUEST_TIERS[Math.min(batchIndex,QUEST_TIERS.length-1)];const rng=seededRng(batchIndex*997+42);const unlocked=COSMETICS.filter(c=>xp>=c.unlockXp);const pool=(unlocked.length>=3?unlocked:COSMETICS).map(c=>c.id);const shuffled=[...pool].sort(()=>rng()-.5);const picked=shuffled.slice(0,3);const tgts=[...tier.targets].sort(()=>rng()-.5);return picked.map((shipId,i)=>({shipId,shipName:COSMETICS[shipId].name,target:tgts[i],gems:tier.gems,done:false}));}
function getOrCreateQuestState(){let state=loadQuestState();if(!state){state={batchIndex:0,batch:generateBatch(0),totalCompleted:0};saveQuestState(state);}return state;}
function checkAndAdvanceBatch(state){if(state.batch.every(q=>q.done)){state.batchIndex++;state.batch=generateBatch(state.batchIndex);saveQuestState(state);showToast("🎉 All quests done! New batch unlocked!",3000);renderQuestsTab();}}
function checkQuestsAfterRun(){const state=getOrCreateQuestState();let anyCompleted=false;state.batch.forEach(q=>{if(q.done)return;if(questRunScore>=q.target&&chosenCosmetic===q.shipId){q.done=true;state.totalCompleted++;gems+=q.gems;saveProgress();updateAllRankUi();showToast(`✅ Quest: Score ${q.target} with ${q.shipName} +${q.gems}💎`,3000);anyCompleted=true;}});if(anyCompleted){saveQuestState(state);checkAndAdvanceBatch(state);}}
function renderQuestsTab(){
  const list=document.getElementById("questsList");list.innerHTML="";
  const state=getOrCreateQuestState();const doneCount=state.batch.filter(q=>q.done).length;const tier=QUEST_TIERS[Math.min(state.batchIndex,QUEST_TIERS.length-1)];
  document.getElementById("questsHeader").innerHTML=`Batch ${state.batchIndex+1} · ${doneCount}/3 complete · Each quest: ${tier.gems}💎<br>Complete all 3 to unlock the next batch`;
  state.batch.forEach((q,i)=>{const sc=COSMETICS[q.shipId];const prog=Math.min(q.target,questRunScore);const pct=q.done?100:Math.min(100,prog/q.target*100);const rarity=i===2?"epic":i===1?"rare":"common";const d=document.createElement("div");d.className=`questCard questRarity-${rarity}${q.done?" done":""}`;d.innerHTML=`<div class="qr">${q.gems}💎</div><div class="qt">Score ${q.target.toLocaleString()} with ${sc.name}</div><div class="qd">${sc.powerDesc}</div>${q.done?`<div class="qdone">✅ Completed!</div>`:`<div class="qbar"><div class="qfill" style="width:${pct}%"></div></div><div class="qprog">${chosenCosmetic===q.shipId?prog:0} / ${q.target} ${chosenCosmetic!==q.shipId?"(select this ship)":""}</div>`}`;list.appendChild(d);});
  const note=document.createElement("div");note.style.cssText="font-size:10px;color:#444;margin-top:10px;text-align:center";note.innerText=`Total quests completed: ${state.totalCompleted}`;list.appendChild(note);
}

document.getElementById("playerNameInput").addEventListener("input",function(){playerName=this.value;saveProgress();});

// HOME SHIP CARD
function updateHomeShipCard(){
  const sc=COSMETICS[chosenCosmetic];
  document.getElementById("hmShipSvg").innerHTML=svgByShipList[sc.id]||"";
  document.getElementById("hmShipName").innerText=sc.name;
  let pt=sc.powerName;if(isBoosted(sc.id))pt="⚡ BOOSTED: "+getBoostedDesc(sc.id).split("—")[0].trim();
  document.getElementById("hmShipPower").style.color=sc.galaxy===2?"#ff88ff":"#00ffcc";
  document.getElementById("hmShipPower").innerText=pt;
  document.getElementById("hmShipCard").onclick=()=>showScreen('hangarScreen');
}

// SVG LIST
const svgByShipList=[
  `<svg viewBox="-22 -30 44 68" width="60" height="60"><polygon points="0,-28 -18,28 18,28" fill="#00ffcc" opacity="0.9"/><rect x="-4" y="-4" width="8" height="16" fill="#00aaaa" rx="2"/><polygon points="-18,28 -22,38 -12,30" fill="#00ddaa"/><polygon points="18,28 22,38 12,30" fill="#00ddaa"/></svg>`,
  `<svg viewBox="-26 -30 52 68" width="60" height="60"><polygon points="0,-28 -20,20 -10,36 10,36 20,20" fill="#00FF00" opacity="0.9"/><polygon points="-26,8 -20,20 0,0" fill="#00cc00"/><polygon points="26,8 20,20 0,0" fill="#00cc00"/></svg>`,
  `<svg viewBox="-20 -30 40 68" width="60" height="60"><ellipse cx="0" cy="0" rx="14" ry="26" fill="#0099FF" opacity="0.9"/><ellipse cx="-14" cy="4" rx="6" ry="4" fill="#0077cc"/><ellipse cx="14" cy="4" rx="6" ry="4" fill="#0077cc"/></svg>`,
  `<svg viewBox="-22 -32 44 70" width="60" height="60"><polygon points="0,-30 -20,16 0,38 20,16" fill="#FF6600" opacity="0.9"/><polygon points="-20,16 -28,22 -10,18" fill="#cc4400"/><polygon points="20,16 28,22 10,18" fill="#cc4400"/></svg>`,
  `<svg viewBox="-24 -20 48 52" width="60" height="60"><ellipse cx="0" cy="0" rx="20" ry="12" fill="#FF00FF" opacity="0.85"/><ellipse cx="0" cy="0" rx="24" ry="4" fill="none" stroke="#FF00FF" stroke-width="2.5" opacity="0.7"/><ellipse cx="0" cy="-4" rx="10" ry="7" fill="#cc00cc" opacity="0.9"/></svg>`,
  `<svg viewBox="-20 -32 40 68" width="60" height="60"><rect x="-16" y="-16" width="32" height="44" fill="#FFFF00" opacity="0.9" rx="3"/><polygon points="0,-32 -12,-16 12,-16" fill="#ffdd00"/><rect x="-20" y="-4" width="40" height="10" fill="#cccc00" rx="2"/></svg>`,
  `<svg viewBox="-26 -34 52 72" width="60" height="60"><ellipse cx="0" cy="0" rx="16" ry="26" fill="#00fff7" opacity="0.9"/><polygon points="0,-34 -18,8 18,8" fill="#00ddee" opacity="0.8"/><polygon points="-16,4 -26,20 -6,24 -12,10" fill="#ffffff" opacity="0.6"/><polygon points="16,4 26,20 6,24 12,10" fill="#ffffff" opacity="0.6"/></svg>`,
  `<svg viewBox="-26 -36 52 76" width="60" height="60"><defs><radialGradient id="vg7b" cx="50%" cy="50%" r="50%"><stop offset="0%" stop-color="#ffffff" stop-opacity="0.9"/><stop offset="100%" stop-color="#ff77ff" stop-opacity="0.2"/></radialGradient></defs><polygon points="0,-34 -13,24 13,24" fill="#ff77ff" opacity="0.92"/><polygon points="-13,12 -26,28 -4,30" fill="#cc44cc" opacity="0.8"/><polygon points="13,12 26,28 4,30" fill="#cc44cc" opacity="0.8"/><circle cx="0" cy="-4" r="7" fill="url(#vg7b)" opacity="0.95"/></svg>`,
  `<svg viewBox="-28 -40 56 82" width="60" height="60"><defs><radialGradient id="sg8b" cx="50%" cy="50%" r="50%"><stop offset="0%" stop-color="#ffffff" stop-opacity="1"/><stop offset="100%" stop-color="#ffd700" stop-opacity="0.1"/></radialGradient></defs><polygon points="0,-37 -11,18 0,32 11,18" fill="#fffacd" opacity="0.93"/><polygon points="-11,4 -28,22 -10,30 -4,20" fill="#e8d880" opacity="0.82"/><polygon points="11,4 28,22 10,30 4,20" fill="#e8d880" opacity="0.82"/><ellipse cx="0" cy="-16" rx="20" ry="5.5" fill="none" stroke="#fffacd" stroke-width="2" opacity="0.95"/><circle cx="0" cy="-10" r="8" fill="url(#sg8b)" opacity="1"/><circle cx="0" cy="-10" r="3.5" fill="#ffffff" opacity="0.95"/></svg>`,
  `<svg viewBox="-28 -38 56 80" width="60" height="60"><defs><radialGradient id="cg9b" cx="50%" cy="50%" r="50%"><stop offset="0%" stop-color="#ffaaaa" stop-opacity="1"/><stop offset="100%" stop-color="#ff3333" stop-opacity="0.1"/></radialGradient></defs><polygon points="0,-36 -12,18 0,30 12,18" fill="#ff6666" opacity="0.9"/><polygon points="-12,8 -28,24 -8,28 -4,16" fill="#cc3333" opacity="0.85"/><polygon points="12,8 28,24 8,28 4,16" fill="#cc3333" opacity="0.85"/><ellipse cx="0" cy="-6" rx="16" ry="4.5" fill="none" stroke="#ff4444" stroke-width="2.2" opacity="0.95"/><circle cx="0" cy="-6" r="7" fill="url(#cg9b)" opacity="1"/><circle cx="0" cy="-6" r="3" fill="#110000" opacity="1"/></svg>`,
  `<svg viewBox="-28 -42 56 86" width="60" height="60"><defs><radialGradient id="ag10b" cx="50%" cy="50%" r="50%"><stop offset="0%" stop-color="#ffffff" stop-opacity="1"/><stop offset="100%" stop-color="#c8c8ff" stop-opacity="0.1"/></radialGradient></defs><polygon points="0,-38 -11,18 0,32 11,18" fill="#c8c8ff" opacity="0.9"/><polygon points="-11,4 -28,24 -10,32 -3,18" fill="#aaaaff" opacity="0.78"/><polygon points="11,4 28,24 10,32 3,18" fill="#aaaaff" opacity="0.78"/><ellipse cx="0" cy="-14" rx="18" ry="5" fill="none" stroke="#aaaaff" stroke-width="2" opacity="0.9"/><circle cx="0" cy="-8" r="8.5" fill="url(#ag10b)" opacity="1"/><circle cx="0" cy="-8" r="3.5" fill="#ffffff" opacity="1"/></svg>`,
  `<svg viewBox="-28 -44 56 90" width="60" height="60"><polygon points="0,-2 -28,32 -14,42 -4,22" fill="#dd44ff" opacity="0.62"/><polygon points="0,-2 28,32 14,42 4,22" fill="#dd44ff" opacity="0.62"/><polygon points="0,-42 -7,18 0,30 7,18" fill="#dd44ff" opacity="0.95"/></svg>`,
  `<svg viewBox="-30 -38 60 82" width="60" height="60"><rect x="-14" y="-20" width="28" height="46" fill="#00ffaa" opacity="0.9" rx="3"/><polygon points="0,-38 -12,-20 12,-20" fill="#00ffaa"/><rect x="-30" y="-6" width="10" height="28" fill="#00cc88" opacity="0.85" rx="2"/><rect x="20" y="-6" width="10" height="28" fill="#00cc88" opacity="0.85" rx="2"/></svg>`,
  `<svg viewBox="-28 -46 56 94" width="60" height="60"><defs><radialGradient id="og13b" cx="50%" cy="50%" r="50%"><stop offset="0%" stop-color="#ffffff" stop-opacity="1"/><stop offset="60%" stop-color="#ff8800" stop-opacity="0.7"/><stop offset="100%" stop-color="#ff8800" stop-opacity="0.1"/></radialGradient></defs><polygon points="-8,4 -28,30 -10,42 -2,20" fill="#ff8800" opacity="0.82"/><polygon points="8,4 28,30 10,42 2,20" fill="#ff8800" opacity="0.82"/><polygon points="0,-44 -10,16 0,28 10,16" fill="#ff8800" opacity="0.92"/><circle cx="0" cy="-10" r="11" fill="url(#og13b)"/><circle cx="0" cy="-10" r="4.5" fill="#ffffff" opacity="1"/></svg>`,
  // Stellar Sovereign (id 14) — chromatic ship
  `<svg viewBox="-30 -44 60 92" width="60" height="60"><defs><radialGradient id="sov14" cx="50%" cy="50%" r="50%"><stop offset="0%" stop-color="#ffffff" stop-opacity="1"/><stop offset="40%" stop-color="#cc44ff" stop-opacity="0.8"/><stop offset="100%" stop-color="#ff44bb" stop-opacity="0.1"/></radialGradient></defs><polygon points="0,-42 -9,12 0,26 9,12" fill="#cc44ff" opacity="0.95"/><polygon points="-9,4 -30,28 -12,38 -2,18" fill="#ff44bb" opacity="0.78"/><polygon points="9,4 30,28 12,38 2,18" fill="#ff44bb" opacity="0.78"/><polygon points="-9,0 -22,20 -6,24" fill="#ffcc00" opacity="0.55"/><polygon points="9,0 22,20 6,24" fill="#ffcc00" opacity="0.55"/><ellipse cx="0" cy="-16" rx="19" ry="5" fill="none" stroke="#cc44ff" stroke-width="2" opacity="0.95"/><ellipse cx="0" cy="-16" rx="25" ry="7" fill="none" stroke="#ff44bb" stroke-width="1" opacity="0.5"/><circle cx="0" cy="-8" r="10" fill="url(#sov14)"/><circle cx="0" cy="-8" r="4" fill="#ffffff" opacity="1"/></svg>`,
];

// HANGAR 3D PREVIEW
let previewRenderer=null,previewScene=null,previewCamera=null,previewShipGroup=null,previewAnimId=null,hangarSelectedId=0;
function initPreviewScene(){
  const canvas=document.getElementById("shipPreviewCanvas");
  if(previewRenderer)previewRenderer.dispose();
  previewScene=new THREE.Scene();previewScene.background=new THREE.Color(0x040c1e);
  previewCamera=new THREE.PerspectiveCamera(50,1,0.1,100);previewCamera.position.set(0,2.5,9);previewCamera.lookAt(0,0,0);
  previewRenderer=new THREE.WebGLRenderer({canvas,antialias:true});previewRenderer.setPixelRatio(Math.min(window.devicePixelRatio,2));previewRenderer.setSize(180,180);
  previewScene.add(new THREE.AmbientLight(0x223355,3));
  const dl=new THREE.DirectionalLight(0x88aaff,2);dl.position.set(-3,5,4);previewScene.add(dl);
  const rl=new THREE.DirectionalLight(0x00ffcc,1.2);rl.position.set(4,-2,-5);previewScene.add(rl);
  const bl=new THREE.PointLight(0xffffff,1.5,20);bl.position.set(0,3,2);previewScene.add(bl);
}
function loadPreviewShip(id){
  if(!previewScene)initPreviewScene();
  if(previewShipGroup){previewScene.remove(previewShipGroup);previewShipGroup=null;}
  if(COSMETICS[id].galaxy===2){previewScene.background.set(0x0d0020);const pl=new THREE.PointLight(0xdd44ff,1.5,18);pl.position.set(0,3,0);previewScene.add(pl);}
  else if(COSMETICS[id].stellarPassOnly){previewScene.background.set(0x08020f);const pl=new THREE.PointLight(0xcc44ff,1.8,18);pl.position.set(0,3,0);previewScene.add(pl);}
  else previewScene.background.set(0x040c1e);
  previewShipGroup=createShipMesh(id);previewShipGroup.rotation.x=0.18;previewScene.add(previewShipGroup);startPreviewAnim();
}
function startPreviewAnim(){
  if(previewAnimId)cancelAnimationFrame(previewAnimId);
  const spin=()=>{if(!previewShipGroup||!previewRenderer)return;previewShipGroup.rotation.y+=0.018;(previewShipGroup.userData.wraithOrbs||[]).forEach((o,i)=>{const t=Date.now()*0.002+i*(Math.PI*2/3);o.position.set(Math.cos(t)*1.6,Math.sin(t*0.7)*0.4+0.2,Math.sin(t)*0.8-0.5);});previewRenderer.render(previewScene,previewCamera);previewAnimId=requestAnimationFrame(spin);};spin();
}
function stopPreviewScene(){if(previewAnimId){cancelAnimationFrame(previewAnimId);previewAnimId=null;}}

// HANGAR LIST & DETAIL
function refreshHangar(){hangarSelectedId=chosenCosmetic;buildHangarList();showHangarDetail(hangarSelectedId);initPreviewScene();loadPreviewShip(hangarSelectedId);}
function buildHangarList(){
  const list=document.getElementById("hangarList");list.innerHTML="";
  const g2Unlocked=xp>=20000;
  COSMETICS.forEach(sc=>{
    const g2=sc.galaxy===2;
    if(g2&&!g2Unlocked)return;
    if(sc.id===11){const sep=document.createElement("div");sep.className="hListSep";sep.innerText="🌌 G2";list.appendChild(sep);}
    const unlocked=xp>=sc.unlockXp;
    const div=document.createElement("div");
    div.className="hListItem"+(sc.id===hangarSelectedId?" selected":"")+(g2?" g2item":"")+(unlocked?"":" locked");
    div.innerHTML=`<div>${svgByShipList[sc.id]||""}</div><div class="hListItemName">${sc.name}</div>`;
    if(isBoosted(sc.id)){const b=document.createElement("span");b.className="hListBoostBadge";b.innerText="⚡";div.appendChild(b);}
    if(g2&&unlocked){const b2=document.createElement("span");b2.className="hListG2Badge";b2.innerText="G2";div.appendChild(b2);}
    // FIX: locked ships don't respond to clicks
    div.onclick=()=>{if(!unlocked)return;hangarSelectedId=sc.id;buildHangarList();showHangarDetail(sc.id);loadPreviewShip(sc.id);};
    list.appendChild(div);
  });
}
function showHangarDetail(id){
  const sc=COSMETICS[id],isPassShip=sc.stellarPassOnly,g2=sc.galaxy===2,unlocked=isShipUnlocked(id);
  document.getElementById("hdName").innerText=sc.name;document.getElementById("hdName").style.color=isPassShip?"#cc44ff":g2?"#ff88ff":"#fff";
  const gEl=document.getElementById("hdGalaxy");
  if(isPassShip){gEl.innerHTML=`<span style="animation:chromaticShift 2.5s linear infinite;display:inline-block;font-weight:bold">✦ STELLAR PASS EXCLUSIVE</span>`;gEl.style.color="";}
  else if(g2){gEl.innerText="🌌 NEBULA RIFT GALAXY";gEl.style.color="#dd44ff";}
  else{gEl.innerText="";}
  document.getElementById("hdPower").innerText=sc.powerName;document.getElementById("hdPower").style.color=g2?"#ff88ff":"#00ffcc";
  document.getElementById("hdPowerDesc").innerText=sc.powerDesc;
  const uEl=document.getElementById("hdUnlock");
  if(unlocked)uEl.innerHTML=`<span style="color:#00cc66">✓ Unlocked</span>`;
  else uEl.innerHTML=`<span style="color:#888">🔒 Requires ${sc.unlockXp.toLocaleString()} XP</span> <span style="color:#556">(you have ${xp.toLocaleString()})</span>`;
  const bdEl=document.getElementById("hdBoostDesc");if(isBoosted(id)){bdEl.style.display="block";bdEl.innerHTML=`⚡ BOOSTED: ${getBoostedDesc(id)}`;}else bdEl.style.display="none";
  // Skin info in hangar
  const owned=loadOwnedSkins();const equippedSkins=loadEquippedSkins();
  const mySkin=SKINS.find(s=>s.shipId===id);
  let skinHtml="";
  if(mySkin){
    const isOwned=owned.includes(mySkin.id);
    const isEquipped=equippedSkins[id]===mySkin.id;
    if(isEquipped){skinHtml=`<div style="font-size:11px;background:#071409;border:1px solid #00ff8844;border-radius:6px;padding:5px 10px;margin-bottom:6px;color:#00ff88;text-align:center">🎨 Skin: <strong>${mySkin.name}</strong> &nbsp;<button onclick="unequipSkin(${id})" style="font-size:10px;background:#2a0808;border:1px solid #ff4444aa;color:#ff6666;border-radius:4px;padding:1px 7px;cursor:pointer">Remove</button></div>`;}
    else if(isOwned){skinHtml=`<div style="font-size:11px;background:#070f18;border:1px solid #44aa6644;border-radius:6px;padding:5px 10px;margin-bottom:6px;color:#88ffaa;text-align:center">🎨 Own: <strong>${mySkin.name}</strong> &nbsp;<button onclick="equipSkin(${id},${mySkin.id})" style="font-size:10px;background:linear-gradient(135deg,#003311,#004d1a);border:1px solid #00cc44;color:#00ff66;border-radius:4px;padding:1px 7px;cursor:pointer">Equip</button></div>`;}
    else{skinHtml="";}
  }
  // Update skin info in the dedicated placeholder — no duplicate accumulation
  document.getElementById("hdSkinInfo").innerHTML=skinHtml;
  const selBtn=document.getElementById("hdSelectBtn");
  if(!unlocked){selBtn.innerText="🔒 Locked";selBtn.className="";selBtn.disabled=true;}
  else if(id===chosenCosmetic){selBtn.innerText="✓ Selected";selBtn.className="isSelected";selBtn.disabled=true;}
  else{selBtn.innerText="✓ Select Ship";selBtn.className="";selBtn.disabled=false;}
  selBtn.onclick=()=>{if(!unlocked||id===chosenCosmetic)return;chosenCosmetic=id;saveProgress();buildHangarList();showHangarDetail(id);updateHomeShipCard();showToast("Ship selected: "+sc.name,1500);};
}
function drawHomeCosmetics(){}
function drawPowerDesc(){}

// CLEAR OBJECTS
function clearGameObjects(){
  asteroids.forEach(a=>{if(a.mesh)scene.remove(a.mesh)});asteroids=[];
  particles3D.forEach(p=>scene.remove(p.mesh));particles3D=[];
  voidParticles.forEach(p=>scene.remove(p.mesh));voidParticles=[];
  soulParticles.forEach(p=>scene.remove(p.mesh));soulParticles=[];
  engineExhaust.forEach(p=>scene.remove(p.mesh));engineExhaust=[];
  clearGems();
  if(laserBeam){scene.remove(laserBeam);laserBeam=null;}
  if(shipGroup){scene.remove(shipGroup);shipGroup=null;}
  if(shieldMesh){scene.remove(shieldMesh);shieldMesh=null;}
  if(ghostDecoy){scene.remove(ghostDecoy);ghostDecoy=null;ghostDecoyLane=-1;ghostDecoyFrames=0;}
  isTeleporting=false;teleportFrames=0;
  voidLight.intensity=0;soulLight.intensity=0;singLight.intensity=0;chronosLight.intensity=0;
  seraphHaloRing=null;seraphSoulBurstFlashFrames=0;chronosGhostMesh=null;chronosPhased=false;chronosTimer=0;singularityTimer=0;darkPulseCounter=0;
}

// SCREEN NAVIGATION
const ALL_SCREENS=['homeMain','hangarScreen','boostsScreen','questsScreen','shopScreen','passScreen'];
function showScreen(id){
  ALL_SCREENS.forEach(s=>{const el=document.getElementById(s);el.style.display='none';});
  const el=document.getElementById(id);
  el.style.display=(id==='hangarScreen')?'flex':'block';
  if(id==='hangarScreen')refreshHangar();
  if(id==='boostsScreen')renderBoostsTab();
  if(id==='questsScreen')renderQuestsTab();
  if(id==='shopScreen')renderShop();
  if(id==='passScreen')renderPass();
}

function showHomescreen(){
  if(animationFrameId!==null){cancelAnimationFrame(animationFrameId);animationFrameId=null;}
  clearGameObjects();applyGalaxyVisuals(false);
  ALL_SCREENS.forEach(s=>document.getElementById(s).style.display='none');  document.getElementById('homeMain').style.display='block';
  document.getElementById("ui").style.display="none";
  document.getElementById("laserDisplay").style.display="none";
  document.getElementById("godlyCooldown").style.display="none";
  document.getElementById("soulBurstDisplay").style.display="none";
  document.getElementById("darkPulseDisplay").style.display="none";
  document.getElementById("singularityDisplay").style.display="none";
  document.getElementById("chronosDisplay").style.display="none";
  document.getElementById("galaxyHUD").style.display="none";
  document.getElementById("controls-hint").style.display="none";
  document.getElementById("returnHomeBtn").style.display="none";
  document.getElementById("mobile-btns").style.display="none";
  updateMobileToggleUI();updateAllRankUi();updateHomeShipCard();
  if(idleAnimId)cancelAnimationFrame(idleAnimId);
  const idleAnimate=()=>{updateStarfield(3);renderer.render(scene,camera);idleAnimId=requestAnimationFrame(idleAnimate);};
  idleAnimate();
}
function hideHomescreen(){
  if(idleAnimId){cancelAnimationFrame(idleAnimId);idleAnimId=null;}
  stopPreviewScene();
  ALL_SCREENS.forEach(s=>document.getElementById(s).style.display='none');
  document.getElementById("ui").style.display="block";
  document.getElementById("controls-hint").style.display="block";
  if(mobileControls)document.getElementById("mobile-btns").style.display="block";
}

// GAME RESET
function resetGame(){
  if(animationFrameId!==null){cancelAnimationFrame(animationFrameId);animationFrameId=null;}
  clearGameObjects();
  baseSpeed=4;score=0;boostKey=false;difficultyTimer=0;shieldActive=false;xpPassiveTimer=0;gameOver=false;
  dodgedCount=0;legendRevives=0;frozenFrames=0;godlyCooldown=0;questRunScore=0;
  laserShots=0;laserFrames3D=0;laserLane3D=0;seraphWingGlow=0;seraphSoulBurstFlashFrames=0;
  singularityTimer=0;chronosTimer=0;chronosPhased=false;darkPulseCounter=0;gemSpawnTimer=0;chromaticSurgeCounter=0;
  loadProgress();
  const g2=isGalaxy2();applyGalaxyVisuals(g2);
  const hudEl=document.getElementById("galaxyHUD");
  hudEl.style.animation="";hudEl.style.color="#ee88ff";hudEl.innerText="🌌 NEBULA RIFT";
  hudEl.style.display=g2?"block":"none";
  const boostedCadet=(chosenCosmetic===0&&isBoosted(0));startShieldActive=boostedCadet;startShieldFrames=boostedCadet?600:0;
  if(hasRunBoost("iron_shell"))shieldActive=true;
  // Also consume any free pass boosts
  const freeBoosts=JSON.parse(localStorage.getItem("sd2_free_boosts")||"[]");
  if(freeBoosts.length>0){
    const used=freeBoosts.shift();
    localStorage.setItem("sd2_free_boosts",JSON.stringify(freeBoosts));
    if(used==="iron_shell")shieldActive=true;
    else if(used==="xp_surge"&&!activeRunBoosts.includes("xp_surge"))activeRunBoosts.push("xp_surge");
    else if(used==="null_field"&&!activeRunBoosts.includes("null_field"))activeRunBoosts.push("null_field");
    showToast(`✦ Pass reward applied: ${RUN_BOOSTS.find(b=>b.id===used)?.name||used}`,1800);
  }
  ship.moveSpeed=hasRunBoost("hyperdrive")?.22:.11;
  laserShots=hasRunBoost("laser_shot")?3:0;
  activeRunBoosts=[];saveProgress();
  ship.lane=1;ship.x=LANES_3D[1];ship.targetX=LANES_3D[1];ship.tilt=0;
  highScore=parseInt(localStorage.getItem("sd2_hs")||"0")||0;
  document.getElementById("score").innerText=0;document.getElementById("highScore").innerText=highScore;
  document.getElementById("gameOver").style.display="none";document.getElementById("returnHomeBtn").style.display="none";
  document.getElementById("reviveFlash").style.display="none";document.getElementById("godlyCooldown").style.display="none";
  const ld=document.getElementById("laserDisplay");if(laserShots>0){ld.style.display="block";document.getElementById("laserLeft").innerText=laserShots;}else ld.style.display="none";
  const sd=document.getElementById("soulBurstDisplay");if(isSeraph()){sd.style.display="block";document.getElementById("soulBurstCount").innerText=getSoulBurstEvery();}else sd.style.display="none";
  const dpd=document.getElementById("darkPulseDisplay");if(isWraith()){dpd.style.display="block";document.getElementById("darkPulseCount").innerText=getDarkPulseEvery();}else dpd.style.display="none";
  document.getElementById("singularityDisplay").style.display=isHarbinger()?"block":"none";
  document.getElementById("chronosDisplay").style.display=isAeon()?"block":"none";
  if(isHarbinger())document.getElementById("singularityTimer").innerText="15";
  if(isAeon())document.getElementById("chronosLabel").innerText="Solid 8s";
  if(isOmega()||getPowerActive("Time Annihilate"))document.getElementById("godlyCooldown").style.display="block";
  shipGroup=createShipMesh(chosenCosmetic);shipGroup.position.set(LANES_3D[1],0,0);scene.add(shipGroup);
  shieldMesh=createShieldMesh();shieldMesh.position.set(LANES_3D[1],0,0);scene.add(shieldMesh);
  engineLight.color.set(COSMETICS[chosenCosmetic].color);
  if(isSovereign())engineLight.color.setHSL(0.8,1,0.6);
  if(isPhantomatic())voidLight.color.set(0xff44ff);if(isSeraph())soulLight.color.set(0xfffacd);
  if(isHarbinger())singLight.color.set(0xff3333);if(isAeon())chronosLight.color.set(0x8888ff);
  if(isWraith())voidLight.color.set(0xdd44ff);if(isOmega())singLight.color.set(0xff8800);
  updateAllRankUi();renderBoostsTab();
}

document.getElementById("returnHomeBtn").onclick=function(){showHomescreen();};

// LASER
function fireLaser(){
  if(laserShots<=0||gameOver)return;laserShots--;laserLane3D=ship.lane;showLaserBeam(LANES_3D[ship.lane]);
  for(let i=asteroids.length-1;i>=0;i--){if(asteroids[i].x===LANES_3D[ship.lane]){scene.remove(asteroids[i].mesh);asteroids.splice(i,1);}}
  document.getElementById("laserLeft").innerText=laserShots;if(laserShots<=0)document.getElementById("laserDisplay").style.display="none";
  showToast("🔥 LASER FIRED"+(laserShots>0?" ("+laserShots+" left)":"— all shots used!"),1200);
}

// ASTEROIDS
function spawnAsteroid(){
  const count=Math.random()<.8?1:2,shuffled=[...LANES_3D].sort(()=>Math.random()-.5);let spawned=0;
  for(let i=0;i<shuffled.length&&spawned<count;i++){const lx=shuffled[i];if(!asteroids.find(a=>a.x===lx&&a.z<SPAWN_Z+30)){const mesh=createAsteroidMesh();mesh.position.set(lx,0,SPAWN_Z);scene.add(mesh);asteroids.push({x:lx,z:SPAWN_Z,mesh,rotX:(Math.random()-.5)*0.04,rotY:(Math.random()-.5)*0.04});spawned++;}}
}
function preventImpossibleRows(){
  const tops=LANES_3D.map(lx=>asteroids.filter(a=>a.x===lx&&a.z<SPAWN_Z+40));
  if(tops.every(arr=>arr.length>0)){const li=Math.floor(Math.random()*3),ar=tops[li][0];if(ar){const idx=asteroids.indexOf(ar);if(idx>-1){scene.remove(ar.mesh);asteroids.splice(idx,1);}}}
}
function getAsteroidSpeedForLane(laneX){let m=getAsteroidSpeedMult();if(isColossus()&&frozenFrames===0){const adj=isBoosted(12)?0.6:0.75;if(laneX!==LANES_3D[ship.lane])m*=adj;}return m;}
function updateAsteroids3D(speed3D){
  for(let index=asteroids.length-1;index>=0;index--){
    const a=asteroids[index];const aspd=frozenFrames>0?0:speed3D*getAsteroidSpeedForLane(a.x);
    a.z+=aspd;if(a.mesh){a.mesh.position.z=a.z;a.mesh.rotation.x+=a.rotX;a.mesh.rotation.y+=a.rotY;}
    if(a.z>PASS_Z){
      scene.remove(a.mesh);asteroids.splice(index,1);
      if(!gameOver){
        score+=10;questRunScore=score;document.getElementById("score").innerText=score;
        addXp(XP_PER_ASTEROID);dodgedCount++;
        const every=getShieldEveryDodged();if(every&&dodgedCount%every===0)shieldActive=true;
        while(score>=xpMilestoneNext){addXp(Math.floor(XP_PER_MILESTONE*getMilestoneXpBonus()));xpMilestoneNext+=XP_MILESTONE_STEP;}
        saveProgress();if(score>highScore){highScore=score;localStorage.setItem("sd2_hs",highScore);document.getElementById("highScore").innerText=highScore;}
        if(isSeraph()&&dodgedCount>0&&dodgedCount%getSoulBurstEvery()===0)triggerSoulBurst();
        if(isSeraph()){const rem=getSoulBurstEvery()-(dodgedCount%getSoulBurstEvery());document.getElementById("soulBurstCount").innerText=rem===getSoulBurstEvery()?getSoulBurstEvery():rem;}
        if(isWraith()){darkPulseCounter++;const dpE=getDarkPulseEvery();document.getElementById("darkPulseCount").innerText=dpE-(darkPulseCounter%dpE);if(darkPulseCounter%dpE===0)triggerDarkPulse();}
        // Chromatic Surge — Stellar Sovereign
        if(isSovereign()){
          chromaticSurgeCounter++;
          if(chromaticSurgeCounter%10===0){
            const surge=Math.floor(Math.random()*4);
            if(surge===0){shieldActive=true;showToast("✦ Chromatic Surge: SHIELD!",1400);}
            else if(surge===1){frozenFrames=120;showToast("✦ Chromatic Surge: FREEZE!",1400);}
            else if(surge===2){const px=ship.x;for(let i=asteroids.length-1;i>=0;i--){if(Math.abs(asteroids[i].x-px)<1){spawnParticles3D(asteroids[i].x,0,asteroids[i].z,5);scene.remove(asteroids[i].mesh);asteroids.splice(i,1);}}showToast("✦ Chromatic Surge: LANE CLEAR!",1400);}
            else{const tx=LANES_3D[ship.lane];gemObjects.forEach(g=>{g.mesh.position.x=tx;g.lane=ship.lane;});showToast("✦ Chromatic Surge: GEM PULL!",1400);}
            singLight.color.setHSL(Math.random(),1,0.5);singLight.intensity=14;
            doScreenFlash("radial-gradient(ellipse at center,rgba(200,100,255,0.6) 0%,rgba(255,100,200,0.3) 50%,transparent 100%)",400);
          }
        }
      }
      continue;
    }
    if(Math.abs(a.x-ship.x)<2.0&&a.z>-2&&a.z<2.5){
      if(isAeon()&&chronosPhased){spawnParticles3D(a.x,0,a.z,5);scene.remove(a.mesh);asteroids.splice(index,1);continue;}
      if(isPhantomatic()&&isTeleporting)continue;
      if(isPhantomatic()&&isBoosted(7)&&ghostDecoy&&ghostDecoyLane===ship.lane){spawnVoidBurst(a.x,0,a.z);scene.remove(ghostDecoy);ghostDecoy=null;ghostDecoyLane=-1;ghostDecoyFrames=0;scene.remove(a.mesh);asteroids.splice(index,1);flashScreen("rgba(200,0,255,.3)");showToast("👻 Ghost Decoy absorbed the hit!",1400);continue;}
      if(chosenCosmetic===0&&isBoosted(0)&&startShieldActive)continue;
      if(shieldActive){shieldActive=false;flashScreen("rgba(255,150,0,.4)");spawnParticles3D(ship.x,0,0,10);scene.remove(a.mesh);asteroids.splice(index,1);continue;}
      if(getPowerActive("Revive")&&legendRevives<getLegendRevives()){legendRevives++;flashScreen("rgba(0,255,200,.3)");spawnParticles3D(a.x,0,a.z,15);scene.remove(a.mesh);asteroids.splice(index,1);continue;}
      spawnParticles3D(ship.x,0,0,25);gameOver=true;
      document.getElementById("gameOver").style.display="block";document.getElementById("returnHomeBtn").style.display="inline-block";
      checkQuestsAfterRun();
      addPassXp(Math.floor(questRunScore/10));
    }
  }
  if(frozenFrames>0)frozenFrames--;if(startShieldActive&&startShieldFrames>0){startShieldFrames--;if(!startShieldFrames)startShieldActive=false;}
}

// SHIP UPDATE
function updateShip3D(){
  if(!isPhantomatic()){ship.x+=(ship.targetX-ship.x)*ship.moveSpeed;ship.tilt=(ship.targetX-ship.x)*.04;}else ship.tilt=0;
  if(shipGroup){
    shipGroup.position.x=ship.x;shipGroup.rotation.z=-ship.tilt;shipGroup.position.y=Math.sin(Date.now()*0.002)*0.12;
    if(isPhantomatic())shipGroup.children.forEach(c=>{if(c.geometry&&c.geometry.type==="TorusGeometry")c.rotation.z+=0.04;});
    if(isSeraph()&&seraphHaloRing){seraphHaloRing.rotation.z+=0.018;const bG=seraphSoulBurstFlashFrames>0?(seraphSoulBurstFlashFrames/45):0;seraphHaloRing.material.opacity=Math.min(1.0,0.7+Math.sin(Date.now()*0.003)*0.15+bG*0.9);seraphWingGlow=0.5+Math.sin(Date.now()*0.0025)*0.3+bG*0.5;if(soulLight.intensity<1){soulLight.intensity=seraphWingGlow*1.8;soulLight.position.set(ship.x,1,-0.5);}}
    if(isHarbinger()){shipGroup.children.forEach(c=>{if(c.geometry&&c.geometry.type==="TorusGeometry")c.rotation.z+=0.03;});singLight.position.set(ship.x,0,0);singLight.intensity=Math.max(0,singLight.intensity*0.92);}
    if(isAeon()){let ri=0;shipGroup.children.forEach(c=>{if(c.geometry&&c.geometry.type==="TorusGeometry"){c.rotation.z+=ri===0?0.022:-0.016;ri++;}});if(chronosGhostMesh){chronosGhostMesh.material.opacity=chronosPhased?0.45+Math.sin(Date.now()*0.01)*0.15:0.16;chronosGhostMesh.position.z=2.8+Math.sin(Date.now()*0.005)*0.3;}chronosLight.position.set(ship.x,0,0);if(chronosPhased){chronosLight.intensity=1.5+Math.sin(Date.now()*0.015)*0.8;if(Math.floor(Date.now()/180)%2===0)shipGroup.visible=Math.random()>0.12;else shipGroup.visible=true;}else{chronosLight.intensity=Math.max(0,chronosLight.intensity*0.92);shipGroup.visible=true;}}
    if(isWraith()){(shipGroup.userData.wraithOrbs||[]).forEach((o,i)=>{const t=Date.now()*0.002+i*(Math.PI*2/3);o.position.set(Math.cos(t)*1.6,Math.sin(t*0.7)*0.4+0.2,Math.sin(t)*0.8-0.5);});voidLight.position.set(ship.x,0,0);voidLight.intensity=Math.max(0,voidLight.intensity*0.92);}
    if(isColossus()){singLight.color.set(0x00ffaa);singLight.position.set(ship.x,0,0);singLight.intensity=Math.max(0,singLight.intensity*0.95);}
    if(isOmega()){let ri=0;shipGroup.children.forEach(c=>{if(c.geometry&&c.geometry.type==="TorusGeometry"){c.rotation.z+=ri===0?0.025:ri===1?-0.018:0.012;ri++;}});singLight.color.set(0xff8800);singLight.position.set(ship.x,0,0);singLight.intensity=Math.max(0,singLight.intensity*0.92);}
    if(isSovereign()){
      // Cycle ring colors chromatically
      const t=Date.now()*0.001;
      let ri=0;
      shipGroup.children.forEach(c=>{
        if(c.geometry&&c.geometry.type==="TorusGeometry"){
          const h=(t*0.3+ri*0.33)%1;
          c.material.color.setHSL(h,1,0.6);
          c.rotation.z+=ri===0?0.028:ri===1?-0.02:0.014;
          ri++;
        }
      });
      // Pulse core orb color
      shipGroup.children.forEach(c=>{
        if(c.geometry&&c.geometry.type==="SphereGeometry"&&c.material.emissiveIntensity>1){
          const h2=(t*0.4)%1;c.material.emissive.setHSL(h2,1,0.5);
        }
      });
      singLight.color.setHSL((t*0.3)%1,1,0.5);
      singLight.position.set(ship.x,0,0);
      singLight.intensity=2.5+Math.sin(t*2)*1;
    }
  }
  const shieldOn=(chosenCosmetic===0&&isBoosted(0)&&startShieldActive)||shieldActive;
  if(shieldMesh){shieldMesh.position.x=ship.x;shieldMesh.position.y=shipGroup?shipGroup.position.y:0;shieldMesh.visible=shieldOn;if(shieldOn){shieldMesh.rotation.y+=0.02;shieldMesh.rotation.x+=0.01;}}
  engineLight.position.set(ship.x,shipGroup?shipGroup.position.y:0,2);engineLight.intensity=1.5+Math.sin(Date.now()*0.01)*0.7+(boostKey?2:0);shipPointLight.position.x=ship.x;
  if(voidLight.intensity>0&&!isWraith())voidLight.intensity=Math.max(0,voidLight.intensity*0.85);
  if(singLight.intensity>0&&!isColossus()&&!isOmega())singLight.intensity=Math.max(0,singLight.intensity*0.92);
  if(Math.random()<0.35&&!isTeleporting)spawnEngineParticle(ship.x,getShipColor(chosenCosmetic));
}

// TIMERS
function updateSingularity(){if(!isHarbinger()||gameOver)return;singularityTimer++;const interval=isBoosted(9)?480:SINGULARITY_INTERVAL;document.getElementById("singularityTimer").innerText=Math.max(0,Math.ceil((interval-singularityTimer)/60));if(singularityTimer>=interval){singularityTimer=0;triggerSingularityPulse();singLight.intensity=10;}}
function updateChronos(){
  if(!isAeon()||gameOver)return;chronosTimer++;const solidTime=isBoosted(10)?300:CHRONOS_SOLID;
  if(!chronosPhased){document.getElementById("chronosLabel").innerText=`Solid ${Math.max(0,Math.ceil((solidTime-chronosTimer)/60))}s`;if(chronosTimer>=solidTime){chronosPhased=true;chronosTimer=0;doChronosFlash(true);showToast("🌀 Chronos Phase — you are intangible!",1600);chronosLight.intensity=3;}}
  else{document.getElementById("chronosLabel").innerText=`PHASED ${Math.max(0,Math.ceil((CHRONOS_PHASED-chronosTimer)/60))}s`;if(chronosTimer>=CHRONOS_PHASED){chronosPhased=false;chronosTimer=0;doChronosFlash(false);showToast("Chronos Shift ended — solid again",1000);if(shipGroup)shipGroup.visible=true;}}
}
function updateGodlyCooldown(){
  const el=document.getElementById("godlyCooldown");
  if(hasAnnihilateAbility()&&!gameOver){el.style.display="block";if(isOmega()){el.style.color="#ff8800";el.style.boxShadow="0 0 8px #ff8800";el.innerText=godlyCooldown>0?`⏳ Omega: ${Math.ceil(godlyCooldown/60)}s`:`Press F — COSMIC RECKONING!`;}else{el.style.color="#00fff7";el.style.boxShadow="0 0 8px #00fff7";el.innerText=godlyCooldown>0?`⏳ Godly: ${Math.ceil(godlyCooldown/60)}s`:`Press F — Time Annihilate!`;}}else if(!hasAnnihilateAbility())el.style.display="none";
}
function updateGemSpawn(){if(gameOver)return;gemSpawnTimer++;const interval=isHarbinger()?300:isGalaxy2()?400:480;if(gemSpawnTimer>=interval){gemSpawnTimer=0;spawnGem(Math.floor(Math.random()*3));}}
function updatePassiveXp(){if(!gameOver){xpPassiveTimer++;if(xpPassiveTimer>=60){addXp(XP_PASSIVE_SEC);xpPassiveTimer=0;}}}
function updateCamera(){camera.position.x+=(ship.x*0.18-camera.position.x)*0.04;camera.lookAt(new THREE.Vector3(ship.x*0.12,-0.5,-20));}

// MAIN LOOP
function update(){
  const spd2d=frozenFrames>0?0:baseSpeed+(baseSpeed*.5*getBoostMultiplier())+(boostKey?4*getBoostMultiplier():0);
  const spd3D=spd2d*SPEED_SCALE;
  updateStarfield(spd2d);
  if(!gameOver){difficultyTimer++;if(frozenFrames===0&&difficultyTimer%300===0)baseSpeed+=.5;updateShip3D();updateAsteroids3D(spd3D);updateGems(spd3D);updateGemSpawn();preventImpossibleRows();if(frozenFrames===0&&Math.random()<.03)spawnAsteroid();if(godlyCooldown>0)godlyCooldown--;updateSingularity();updateChronos();}
  updateLaser3D();updateEngineExhaust();updateParticles3D();updateVoidParticles();updateSoulParticles();updateGhostDecoy();updatePassiveXp();updateGodlyCooldown();updateCamera();
  renderer.render(scene,camera);animationFrameId=requestAnimationFrame(update);
}

// EVENTS
document.getElementById("startBtn").onclick=()=>{hideHomescreen();resetGame();animationFrameId=requestAnimationFrame(update);};
document.getElementById("navHangar").onclick=()=>showScreen('hangarScreen');
document.getElementById("navBoosts").onclick=()=>showScreen('boostsScreen');
document.getElementById("navQuests").onclick=()=>showScreen('questsScreen');
document.getElementById("navPass").onclick=()=>showScreen('passScreen');
document.getElementById("navShop").onclick=()=>showScreen('shopScreen');

// ══════════════════════════════════════════════════════════
// STELLAR PASS SYSTEM
// ══════════════════════════════════════════════════════════
const PASS_TIERS = [
  {tier:1,  xpRequired:50,   reward:{type:"gems",   amount:1,     label:"💎 1 Gem",             desc:"A free gem, just for playing!"}},
  {tier:2,  xpRequired:150,  reward:{type:"boost",  id:"iron_shell",label:"🛡️ Iron Shell Boost", desc:"Start your next run with a shield"}},
  {tier:3,  xpRequired:300,  reward:{type:"gems",   amount:2,     label:"💎 2 Gems",            desc:"Keep it up!"}},
  {tier:4,  xpRequired:500,  reward:{type:"skin",   skinId:14,    label:"🌈 Chromatic Viper",   desc:"Chromatic skin for Cadet Viper"}},
  {tier:5,  xpRequired:800,  reward:{type:"boost",  id:"xp_surge", label:"✨ XP Surge Boost",   desc:"3× XP for one full run"}},
  {tier:6,  xpRequired:1200, reward:{type:"gems",   amount:3,     label:"💎 3 Gems",            desc:"You're halfway there!"}},
  {tier:7,  xpRequired:1800, reward:{type:"boost",  id:"null_field",label:"❄️ Null Field Boost", desc:"Asteroids 35% slower for one run"}},
  {tier:8,  xpRequired:2600, reward:{type:"skin",   skinId:15,    label:"🌈 Chromatic Phantom",  desc:"Chromatic skin for Mythic Phantom"}},
  {tier:9,  xpRequired:3600, reward:{type:"gems",   amount:5,     label:"💎 5 Gems",            desc:"Almost at the top!"}},
  {tier:10, xpRequired:5000, reward:{type:"ship",   shipId:14,    label:"✦ Stellar Sovereign",  desc:"The exclusive Chromatic ship with Chromatic Surge power"}},
];

function loadPassState(){try{return JSON.parse(localStorage.getItem("sd2_pass")||"null");}catch(e){return null;}}
function savePassState(s){localStorage.setItem("sd2_pass",JSON.stringify(s));}
function getOrCreatePassState(){
  let s=loadPassState();
  if(!s)s={passXp:0,claimed:[]};
  return s;
}
function addPassXp(amount){
  if(amount<=0)return;
  const s=getOrCreatePassState();
  s.passXp+=amount;
  savePassState(s);
  // Check if any tier just unlocked
  const justUnlocked=PASS_TIERS.filter(t=>!s.claimed.includes(t.tier)&&s.passXp>=t.xpRequired);
  if(justUnlocked.length>0)showToast(`✦ StellarPass tier ${justUnlocked[justUnlocked.length-1].tier} unlocked! Claim your reward!`,2800);
  if(document.getElementById("passScreen").style.display!=="none")renderPass();
}
function claimPassTier(tier){
  const s=getOrCreatePassState();
  if(s.claimed.includes(tier))return;
  const def=PASS_TIERS.find(t=>t.tier===tier);
  if(!def||s.passXp<def.xpRequired)return;
  s.claimed.push(tier);savePassState(s);
  const r=def.reward;
  if(r.type==="gems"){gems+=r.amount;saveProgress();updateAllRankUi();showToast(`✦ Claimed: ${r.label}!`,1800);}
  else if(r.type==="boost"){
    // Give a free run boost charge stored separately
    const freeBoosts=JSON.parse(localStorage.getItem("sd2_free_boosts")||"[]");
    freeBoosts.push(r.id);localStorage.setItem("sd2_free_boosts",JSON.stringify(freeBoosts));
    showToast(`✦ Claimed: ${r.label}! (auto-applied next run)`,2200);
  }
  else if(r.type==="skin"){
    const owned=loadOwnedSkins();
    if(!owned.includes(r.skinId)){owned.push(r.skinId);saveOwnedSkins(owned);}
    showToast(`✦ Claimed: ${r.label}! (equip it in Shop or Hangar)`,2500);
  }
  else if(r.type==="ship"){
    // Unlock the sovereign ship by storing it
    const unlockedPassShips=JSON.parse(localStorage.getItem("sd2_pass_ships")||"[]");
    if(!unlockedPassShips.includes(r.shipId)){unlockedPassShips.push(r.shipId);localStorage.setItem("sd2_pass_ships",JSON.stringify(unlockedPassShips));}
    showToast(`✦ CLAIMED: ✦ STELLAR SOVEREIGN ✦ — the Chromatic ship is yours!`,3500);
  }
  renderPass();updateAllRankUi();
  if(document.getElementById("hangarScreen").style.display!=="none")refreshHangar();
}
function isPassShipUnlocked(shipId){
  if(shipId!==14)return false;
  try{return JSON.parse(localStorage.getItem("sd2_pass_ships")||"[]").includes(shipId);}catch(e){return false;}
}
// Override COSMETICS unlock check for pass ship
const _origIsUnlocked=id=>xp>=COSMETICS[id].unlockXp;
function isShipUnlocked(id){if(COSMETICS[id].stellarPassOnly)return isPassShipUnlocked(id);return xp>=COSMETICS[id].unlockXp;}

function renderPass(){
  const s=getOrCreatePassState();
  // XP bar
  const maxXp=PASS_TIERS[PASS_TIERS.length-1].xpRequired;
  const pct=Math.min(100,(s.passXp/maxXp)*100);
  document.getElementById("passXpCur").innerText=s.passXp.toLocaleString();
  const nextTier=PASS_TIERS.find(t=>s.passXp<t.xpRequired);
  document.getElementById("passXpNext").innerText=nextTier?`Next tier: ${nextTier.xpRequired.toLocaleString()} XP`:"MAX — all tiers reached!";
  document.getElementById("passXpFill").style.width=pct+"%";
  // Tiers
  const grid=document.getElementById("passTiersGrid");grid.innerHTML="";
  PASS_TIERS.forEach(def=>{
    const unlocked=s.passXp>=def.xpRequired;
    const claimed=s.claimed.includes(def.tier);
    const available=unlocked&&!claimed;
    const r=def.reward;
    const isChromatic=r.type==="skin"||r.type==="ship";
    const card=document.createElement("div");
    card.className="passTier "+(claimed?"tier-claimed":available?"tier-available":unlocked?"tier-unlocked":"tier-locked");
    let rewardIcon="";
    if(r.type==="gems")rewardIcon="💎";
    else if(r.type==="boost")rewardIcon="⚡";
    else if(r.type==="skin")rewardIcon="🌈";
    else if(r.type==="ship")rewardIcon="✦";
    let actionHtml="";
    if(claimed)actionHtml=`<span class="tierClaimedBadge">✓ Claimed</span>`;
    else if(available)actionHtml=`<button class="tierClaimBtn" onclick="claimPassTier(${def.tier})">Claim!</button>`;
    else actionHtml=`<span class="tierLockedBadge">🔒</span>`;
    const rewardNameStyle=isChromatic?`style="animation:chromaticShift 2.5s linear infinite;display:inline-block"`:"";
    card.innerHTML=`
      <div class="tierNum">${def.tier}</div>
      <div class="tierReward">
        <div class="tierRewardName" ${rewardNameStyle}>${rewardIcon} ${r.label}</div>
        <div class="tierRewardDesc">${r.desc}</div>
      </div>
      <div class="tierXpReq">${claimed?"":s.passXp>=def.xpRequired?"Ready!":def.xpRequired.toLocaleString()+" XP"}</div>
      ${actionHtml}`;
    grid.appendChild(card);
  });
}

// Patch saveProgress / loadProgress to handle pass ship unlock check
const _origBuildHangarList=buildHangarList;
buildHangarList=function(){
  const list=document.getElementById("hangarList");list.innerHTML="";
  const g2Unlocked=xp>=20000;
  COSMETICS.forEach(sc=>{
    const g2=sc.galaxy===2;
    const isPassShip=sc.stellarPassOnly;
    if(g2&&!g2Unlocked&&!isPassShip)return;
    if(isPassShip&&!isPassShipUnlocked(sc.id))return;
    if(sc.id===11){const sep=document.createElement("div");sep.className="hListSep";sep.innerText="🌌 G2";list.appendChild(sep);}
    if(isPassShip){const sep=document.createElement("div");sep.className="hListSep";sep.style.color="#cc44ff";sep.style.borderTopColor="#330044";sep.style.animation="chromaticShift 2.5s linear infinite";sep.innerText="✦ PASS";list.appendChild(sep);}
    const unlocked=isShipUnlocked(sc.id);
    const div=document.createElement("div");
    // Pass ship gets its own chromatic style, not g2item
    div.className="hListItem"+(sc.id===hangarSelectedId?" selected":"")+(g2&&!isPassShip?" g2item":"")+(isPassShip?" pass-item":"")+(unlocked?"":" locked");
    div.innerHTML=`<div>${svgByShipList[sc.id]||""}</div><div class="hListItemName">${sc.name}</div>`;
    if(isBoosted(sc.id)){const b=document.createElement("span");b.className="hListBoostBadge";b.innerText="⚡";div.appendChild(b);}
    // G2 badge for galaxy-2 ships only, ✦ badge for pass ship only
    if(g2&&!isPassShip&&unlocked){const b2=document.createElement("span");b2.className="hListG2Badge";b2.innerText="G2";div.appendChild(b2);}
    if(isPassShip&&unlocked){const bp=document.createElement("span");bp.className="hListG2Badge";bp.style.background="linear-gradient(135deg,#6600cc,#ff44bb)";bp.style.animation="chromaticShift 2.5s linear infinite";bp.innerText="✦";div.appendChild(bp);}
    div.onclick=()=>{if(!unlocked)return;hangarSelectedId=sc.id;buildHangarList();showHangarDetail(sc.id);loadPreviewShip(sc.id);};
    list.appendChild(div);
  });
};
document.getElementById("hangarBack").onclick=()=>{stopPreviewScene();showScreen('homeMain');updateHomeShipCard();};
document.querySelectorAll(".screenBack").forEach(btn=>btn.onclick=()=>showScreen('homeMain'));
document.getElementById("boosterBtn").onclick=function(){const r=runBoosterLogic();if(r!==undefined){buildHangarList();showHangarDetail(hangarSelectedId);updateHomeShipCard();}};

// ══════════════════════════════════════════════════════════
// SHOP SYSTEM
// ══════════════════════════════════════════════════════════
const SHOP_REFRESH_MS = 20 * 60 * 1000; // 20 minutes

function loadShopState(){
  try{
    const stored = localStorage.getItem("sd2_shop");
    if(stored) return JSON.parse(stored);
  }catch(e){}
  return null;
}
function saveShopState(state){localStorage.setItem("sd2_shop",JSON.stringify(state));}
function loadOwnedSkins(){try{return JSON.parse(localStorage.getItem("sd2_owned_skins")||"[]");}catch(e){return [];}}
function saveOwnedSkins(arr){localStorage.setItem("sd2_owned_skins",JSON.stringify(arr));}
function loadEquippedSkins(){try{return JSON.parse(localStorage.getItem("sd2_equipped_skins")||"{}");}catch(e){return {};}}
function saveEquippedSkins(obj){localStorage.setItem("sd2_equipped_skins",JSON.stringify(obj));}
function getEquippedSkin(shipId){const e=loadEquippedSkins();return e[shipId]!==undefined?e[shipId]:null;}

function generateShopSlots(){
  // Only include skins that are NOT pass-exclusive
  const all=SKINS.filter(s=>!s.passOnly).map(s=>s.id);
  const shuffled=[...all].sort(()=>Math.random()-.5);
  return shuffled.slice(0,3);
}
function getOrCreateShopState(){
  let state=loadShopState();
  const now=Date.now();
  if(!state||now-state.lastRefresh>=SHOP_REFRESH_MS){
    state={slots:generateShopSlots(),lastRefresh:now};
    saveShopState(state);
  }
  return state;
}

function buySkin(skinId){
  const skin=SKINS.find(s=>s.id===skinId);if(!skin)return;
  if(gems<skin.cost){showToast("Not enough 💎 Gems!");return;}
  const owned=loadOwnedSkins();
  if(owned.includes(skinId)){showToast("You already own this skin!");return;}
  gems-=skin.cost;owned.push(skinId);
  saveOwnedSkins(owned);saveProgress();updateAllRankUi();
  showToast(`🎨 Bought: ${skin.name}!`,2000);
  renderShop();
}
function equipSkin(shipId,skinId){
  const equipped=loadEquippedSkins();
  equipped[shipId]=skinId;
  saveEquippedSkins(equipped);
  showToast("✨ Skin equipped!",1200);
  renderShop();updateHomeShipCard();
  // Refresh hangar detail if open
  if(document.getElementById("hangarScreen").style.display!=="none")showHangarDetail(hangarSelectedId);
}
function unequipSkin(shipId){
  const equipped=loadEquippedSkins();
  delete equipped[shipId];
  saveEquippedSkins(equipped);
  showToast("Skin removed.",900);
  renderShop();updateHomeShipCard();
  if(document.getElementById("hangarScreen").style.display!=="none")showHangarDetail(hangarSelectedId);
}

function renderShop(){
  const state=getOrCreateShopState();
  const owned=loadOwnedSkins();
  const equipped=loadEquippedSkins();
  const container=document.getElementById("shopCards");
  container.innerHTML="";

  state.slots.forEach(skinId=>{
    const skin=SKINS.find(s=>s.id===skinId);if(!skin)return;
    const ship=COSMETICS[skin.shipId];
    const isOwned=owned.includes(skinId);
    const isEquipped=equipped[skin.shipId]===skinId;
    const shipUnlocked=xp>=ship.unlockXp;

    const card=document.createElement("div");
    card.className="skinCard "+(isEquipped?"owned-equipped":isOwned?"owned-unequipped":"not-owned");

    // Color swatch with SVG ship preview using skin color
    const svgSrc=svgByShipList[skin.shipId]||"";
    // recolor SVG fills to skin color
    const coloredSvg=svgSrc.replace(/fill="[^"]*"/g,`fill="${skin.color}"`).replace(/stroke="[^"]*"/g,`stroke="${skin.color}"`);
    const hex=skin.color.replace("#","");
    const r=parseInt(hex.slice(0,2),16)/255,g=parseInt(hex.slice(2,4),16)/255,b=parseInt(hex.slice(4,6),16)/255;
    const glowR=Math.round(r*255),glowG=Math.round(g*255),glowB=Math.round(b*255);
    const glowCss=`rgba(${glowR},${glowG},${glowB},0.45)`;

    let actionsHtml="";
    if(isEquipped){
      actionsHtml=`<button class="skinEquipBtn equipped" disabled>✓ Equipped</button><button class="skinUnequipBtn" onclick="unequipSkin(${skin.shipId})">Remove</button>`;
    } else if(isOwned){
      actionsHtml=`<button class="skinEquipBtn" onclick="equipSkin(${skin.shipId},${skinId})">Equip</button>`;
    } else {
      const canAfford=gems>=skin.cost;
      actionsHtml=`<div class="skinCost">${skin.cost} 💎</div><button class="skinBuyBtn" ${(!canAfford||!shipUnlocked)?"disabled":""} onclick="buySkin(${skinId})">${!shipUnlocked?"🔒 Locked":canAfford?"Buy":"Can't afford"}</button>`;
    }

    card.style.setProperty("--sw-glow",glowCss);
    card.innerHTML=`
      ${isOwned?`<div class="skinOwnedBadge">✓ Owned</div>`:""}
      <div class="skinColorSwatch" style="background:radial-gradient(circle at 40% 35%,${skin.color}22,#050a14);box-shadow:inset 0 0 14px #0008,0 0 12px ${glowCss};">${coloredSvg}</div>
      <div class="skinInfo">
        <div class="skinShipName">${ship.name}</div>
        <div class="skinName">${skin.name}</div>
        <span class="skinRarity rarity-${skin.rarity}">${skin.rarity.toUpperCase()}</span>
        ${!shipUnlocked?`<div style="font-size:10px;color:#554">Requires ${ship.unlockXp.toLocaleString()} XP to use</div>`:""}
      </div>
      <div class="skinActions">${actionsHtml}</div>`;
    container.appendChild(card);
  });
}

// SHOP COUNTDOWN TIMER
function updateShopTimer(){
  const state=loadShopState();
  if(!state){document.getElementById("shopTimerVal").innerText="20:00";return;}
  const now=Date.now();
  let ms=SHOP_REFRESH_MS-(now-state.lastRefresh);
  if(ms<=0){
    // Time's up — force refresh
    const newState={slots:generateShopSlots(),lastRefresh:now};
    saveShopState(newState);
    ms=SHOP_REFRESH_MS;
    if(document.getElementById("shopScreen").style.display!=="none")renderShop();
    showToast("🛒 Shop refreshed with new skins!",2500);
  }
  const totalSec=Math.ceil(ms/1000);
  const mins=Math.floor(totalSec/60);
  const secs=totalSec%60;
  document.getElementById("shopTimerVal").innerText=`${mins}:${secs.toString().padStart(2,"0")}`;
}
// Run timer every second
setInterval(updateShopTimer, 1000);
updateShopTimer();
// TUTORIAL
const TUTORIAL_STEPS=[
  {icon:"🚀",title:"Welcome to Stellar Drift!",body:"You're piloting a ship through an asteroid field. Dodge asteroids to survive, earn XP, unlock ships, and collect gems.\n\nLet's learn the basics — it only takes a minute!"},
  {icon:"↔️",title:"Moving Your Ship",body:"Your ship flies in <strong>3 lanes</strong>. Switch lanes to dodge incoming asteroids.\n\n<span class='key'>← →</span> or <span class='key'>A</span> <span class='key'>D</span> on keyboard.\n\nOn mobile, use the <strong>◀ ▶ buttons</strong> at the bottom.",highlight:"controls-hint"},
  {icon:"⚡",title:"Boost",body:"Hold <span class='key'>SPACE</span> (or the <strong>BOOST button</strong> on mobile) to fly faster!\n\nBoost is unlimited, but don't get reckless."},
  {icon:"💥",title:"Asteroids & Shields",body:"Hit an asteroid and it's <strong>Game Over</strong> — unless you have a shield! 🛡️\n\nShields absorb one hit. Equip <strong>Iron Shell</strong> boost to start with one."},
  {icon:"⭐",title:"XP & Ranks",body:"Earn <strong>XP</strong> by dodging asteroids and surviving. Reach new ranks to <strong>unlock new ships</strong>, each with a unique power!",highlight:"homeRank"},
  {icon:"💎",title:"Gems",body:"<strong>💎 Gems</strong>: earn by leveling up and quests. Spend on:\n• <strong>Ship Boosters</strong> — enhance your ship\n• <strong>Run Boosts</strong> — one-run power-ups"},
  {icon:"📜",title:"Quests",body:"Check <strong>Quests</strong> for score challenges tied to specific ships. Complete all 3 in a batch for bonus gems!"},
  {icon:"🌌",title:"Galaxy 2 — Nebula Rift",body:"Reach <strong>Transcendent rank (20,000 XP)</strong> to unlock the Nebula Rift Galaxy — 3 legendary ships with brand new powers!"},
  {icon:"🎮",title:"You're Ready!",body:"• <span class='key'>← →</span> Move &nbsp;|&nbsp; <span class='key'>SPACE</span> Boost\n• <span class='key'>F</span> Ship Ability &nbsp;|&nbsp; <span class='key'>L</span> Laser\n• <span class='key'>R</span> Return Home\n\n<strong>Hit Play and survive as long as you can!</strong>"},
];
let tutStep=0,tutArrowEl=null;
function tutRenderDots(){const d=document.getElementById("tutorialDots");d.innerHTML="";TUTORIAL_STEPS.forEach((_,i)=>{const s=document.createElement("span");s.className="tut-dot"+(i===tutStep?" active":i<tutStep?" done":"");d.appendChild(s);});}
function tutShowStep(idx){
  tutStep=idx;const step=TUTORIAL_STEPS[idx];
  document.getElementById("tutorialIcon").innerText=step.icon;document.getElementById("tutorialTitle").innerText=step.title;
  document.getElementById("tutorialBody").innerHTML=step.body.replace(/\n/g,"<br>");tutRenderDots();
  const nextBtn=document.getElementById("tutorialNext");const isLast=idx===TUTORIAL_STEPS.length-1;nextBtn.innerText=isLast?"🚀 Let's Go!":"Next →";nextBtn.className=isLast?"finish":"";
  if(tutArrowEl){tutArrowEl.remove();tutArrowEl=null;}
  if(step.highlight){const t=document.getElementById(step.highlight);if(t){const r=t.getBoundingClientRect();tutArrowEl=document.createElement("div");tutArrowEl.className="tut-arrow";tutArrowEl.innerText="👆";tutArrowEl.style.left=(r.left+r.width/2-16)+"px";tutArrowEl.style.top=Math.max(4,r.top-44)+"px";document.body.appendChild(tutArrowEl);}}
  const box=document.getElementById("tutorialBox");box.style.animation="none";void box.offsetWidth;box.style.animation="tutSlideIn .25s ease-out";
}
function tutNext(){if(tutStep<TUTORIAL_STEPS.length-1)tutShowStep(tutStep+1);else tutDismiss();}
function tutDismiss(){if(tutArrowEl){tutArrowEl.remove();tutArrowEl=null;}document.getElementById("tutorialOverlay").style.display="none";localStorage.setItem("sd2_tutdone","1");}
function maybeShowTutorial(){if(localStorage.getItem("sd2_tutdone")==="1")return;tutStep=0;tutShowStep(0);document.getElementById("tutorialOverlay").style.display="flex";}
document.getElementById("tutorialNext").addEventListener("click",tutNext);
document.getElementById("tutorialSkip").addEventListener("click",tutDismiss);
document.addEventListener("keydown",e=>{if(document.getElementById("tutorialOverlay").style.display!=="none"){if(e.key==="Enter"||e.key===" "){e.preventDefault();tutNext();}if(e.key==="Escape")tutDismiss();}});

loadProgress();showHomescreen();maybeShowTutorial();

// INPUT
document.addEventListener("keydown",e=>{
  if(document.getElementById("ui").style.display==="none")return;
  if(e.key==="ArrowLeft"||e.key==="a"){if(isPhantomatic()){if(ship.lane>0)doPhantomTeleport(ship.lane-1);}else{if(ship.lane>0){ship.lane--;ship.targetX=LANES_3D[ship.lane];}}}
  if(e.key==="ArrowRight"||e.key==="d"){if(isPhantomatic()){if(ship.lane<2)doPhantomTeleport(ship.lane+1);}else{if(ship.lane<2){ship.lane++;ship.targetX=LANES_3D[ship.lane];}}}
  if(e.key===" "){e.preventDefault();boostKey=true;}
  if(e.key==="r"&&gameOver)showHomescreen();
  if(e.key.toLowerCase()==="l"&&laserShots>0&&!gameOver)fireLaser();
  if(e.key.toLowerCase()==="f"&&!gameOver){if(isOmega()&&godlyCooldown===0){triggerCosmicReckoning();}else if(getPowerActive("Time Annihilate")&&godlyCooldown===0){asteroids.forEach(a=>scene.remove(a.mesh));asteroids=[];godlyCooldown=getGodlyCooldownFrames();frozenFrames=180;flashScreen("rgba(0,255,255,.25)");showToast("⚡ TIME ANNIHILATE!",1500);}}
});
document.addEventListener("keyup",e=>{if(e.key===" ")boostKey=false;});

// MOBILE
function updateMobileToggleUI(){const btn=document.getElementById("mobile-toggle");if(mobileControls){btn.innerText="ON";btn.classList.add("on");}else{btn.innerText="OFF";btn.classList.remove("on");}}
function updateAbilityBtn(){const ab=document.getElementById("mbtn-ability");if(!ab)return;if(laserShots>0){ab.style.display="flex";ab.innerText="🔥";}else if(isOmega()){ab.style.display="flex";ab.innerText=godlyCooldown>0?"⏳":"🔥";}else if(getPowerActive("Time Annihilate")){ab.style.display="flex";ab.innerText=godlyCooldown>0?"⏳":"⚡";}else ab.style.display="none";}
document.getElementById("mobile-toggle").addEventListener("click",function(){mobileControls=!mobileControls;saveProgress();updateMobileToggleUI();if(document.getElementById("ui").style.display!=="none")document.getElementById("mobile-btns").style.display=mobileControls?"block":"none";});
(function setupMobile(){
  function doLeft(){if(document.getElementById("ui").style.display==="none"||gameOver)return;if(isPhantomatic()){if(ship.lane>0)doPhantomTeleport(ship.lane-1);}else if(ship.lane>0){ship.lane--;ship.targetX=LANES_3D[ship.lane];}}
  function doRight(){if(document.getElementById("ui").style.display==="none"||gameOver)return;if(isPhantomatic()){if(ship.lane<2)doPhantomTeleport(ship.lane+1);}else if(ship.lane<2){ship.lane++;ship.targetX=LANES_3D[ship.lane];}}
  function doAbility(){if(gameOver)return;if(laserShots>0){fireLaser();return;}if(isOmega()&&godlyCooldown===0){triggerCosmicReckoning();return;}if(getPowerActive("Time Annihilate")&&godlyCooldown===0){asteroids.forEach(a=>scene.remove(a.mesh));asteroids=[];godlyCooldown=getGodlyCooldownFrames();frozenFrames=180;flashScreen("rgba(0,255,255,.25)");showToast("⚡ TIME ANNIHILATE!",1500);}}
  function addBtn(id,down,up){const el=document.getElementById(id);if(!el)return;el.addEventListener("touchstart",e=>{e.preventDefault();if(down)down();},{passive:false});if(up)el.addEventListener("touchend",e=>{e.preventDefault();up();},{passive:false});if(up)el.addEventListener("touchcancel",()=>up(),{passive:false});el.addEventListener("mousedown",()=>{if(down)down();});if(up)el.addEventListener("mouseup",()=>up());}
  addBtn("mbtn-left",doLeft,null);addBtn("mbtn-right",doRight,null);
  addBtn("mbtn-boost",()=>{boostKey=true;},()=>{boostKey=false;});
  addBtn("mbtn-ability",doAbility,null);
  const _orig=update;update=function(){_orig.apply(this,arguments);if(mobileControls)updateAbilityBtn();};
})();
</script>
</body>
</html>
