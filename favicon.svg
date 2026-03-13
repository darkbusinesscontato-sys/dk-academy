import { useState, useRef, useEffect, useCallback } from "react";
import { AreaChart, Area, XAxis, YAxis, Tooltip, ResponsiveContainer, CartesianGrid, BarChart, Bar, Cell, LineChart, Line } from "recharts";

/* ═══════════════════════════════════════════════════════════════
   DK ACADEMY — Multi-User SaaS with Supabase Auth
   Cada usuário tem dados 100% isolados (RLS)
═══════════════════════════════════════════════════════════════ */

// ── SUPABASE CONFIG ─────────────────────────────────────────────
// Substitua pelos seus dados em: supabase.com > Project Settings > API
const SUPABASE_URL  = "https://heqtqpkdgluvgnuwvljl.supabase.co";
const SUPABASE_KEY  = "sb_publishable_3YZT2nla0XJaSKrzB0XFeg_y-_QmY79";

// ── SUPABASE CLIENT (sem biblioteca externa) ────────────────────
const sb = {
  url: SUPABASE_URL,
  key: SUPABASE_KEY,
  token: null,
  userId: null,

  headers() {
    const h = { "Content-Type": "application/json", "apikey": this.key, "Prefer": "return=representation" };
    if (this.token) h["Authorization"] = `Bearer ${this.token}`;
    return h;
  },

  async signUp(email, password) {
    const r = await fetch(`${this.url}/auth/v1/signup`, {
      method: "POST", headers: { "Content-Type": "application/json", "apikey": this.key },
      body: JSON.stringify({ email, password })
    });
    const d = await r.json();
    if (d.access_token) { this.token = d.access_token; this.userId = d.user?.id; }
    return d;
  },

  async signIn(email, password) {
    const r = await fetch(`${this.url}/auth/v1/token?grant_type=password`, {
      method: "POST", headers: { "Content-Type": "application/json", "apikey": this.key },
      body: JSON.stringify({ email, password })
    });
    const d = await r.json();
    if (d.access_token) { this.token = d.access_token; this.userId = d.user?.id; }
    return d;
  },

  async signOut() {
    await fetch(`${this.url}/auth/v1/logout`, { method: "POST", headers: this.headers() });
    this.token = null; this.userId = null;
  },

  async select(table, query = "") {
    const r = await fetch(`${this.url}/rest/v1/${table}?${query}&order=created_at.desc`, { headers: this.headers() });
    return r.json();
  },

  async insert(table, data) {
    const body = Array.isArray(data) ? data : [{ ...data, user_id: this.userId }];
    if (!Array.isArray(data)) body[0].user_id = this.userId;
    const r = await fetch(`${this.url}/rest/v1/${table}`, {
      method: "POST", headers: this.headers(), body: JSON.stringify(body)
    });
    return r.json();
  },

  async update(table, id, data) {
    const r = await fetch(`${this.url}/rest/v1/${table}?id=eq.${id}`, {
      method: "PATCH", headers: this.headers(), body: JSON.stringify(data)
    });
    return r.json();
  },

  async delete(table, id) {
    await fetch(`${this.url}/rest/v1/${table}?id=eq.${id}`, { method: "DELETE", headers: this.headers() });
  },

  async getProfile() {
    const r = await fetch(`${this.url}/rest/v1/profiles?id=eq.${this.userId}`, { headers: this.headers() });
    const d = await r.json();
    return d[0] || null;
  },

  async updateProfile(data) {
    await fetch(`${this.url}/rest/v1/profiles?id=eq.${this.userId}`, {
      method: "PATCH", headers: this.headers(), body: JSON.stringify(data)
    });
  }
};

/* ══════════ ESTILOS ════════════════════════════════════════════ */
const Styles = () => <style>{`
@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=Orbitron:wght@400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap');
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0;}
:root{
  --dk:#7C3AED;--dk-l:#9D5FF5;--dk-xl:#C084FC;--dk-2xl:#E9D5FF;
  --dk-dim:rgba(124,58,237,0.12);--dk-dim2:rgba(124,58,237,0.06);
  --dk-glow:rgba(124,58,237,0.45);--dk-border:rgba(124,58,237,0.25);
  --neon:#D946EF;--neon-dim:rgba(217,70,239,0.1);
  --bg:#07050E;--s1:#0D0A1A;--s2:#130F22;--s3:#1A1430;--s4:#221B3D;
  --border:rgba(124,58,237,0.14);--border2:rgba(255,255,255,0.06);
  --up:#22C55E;--up-dim:rgba(34,197,94,0.1);
  --dn:#F43F5E;--dn-dim:rgba(244,63,94,0.1);
  --warn:#F59E0B;--warn-dim:rgba(245,158,11,0.1);
  --info:#38BDF8;--info-dim:rgba(56,189,248,0.1);
  --t1:#F0EAFF;--t2:#9B91B8;--t3:#5C5478;--t4:#2E2947;
  --font:'Space Grotesk',sans-serif;--head:'Orbitron',monospace;--mono:'JetBrains Mono',monospace;
  --r:10px;--r2:16px;--r3:24px;
}
html,body,#root{height:100%;background:var(--bg);color:var(--t1);font-family:var(--font);}
body::before{content:'';position:fixed;inset:0;pointer-events:none;z-index:0;
  background-image:linear-gradient(rgba(124,58,237,0.04) 1px,transparent 1px),
    linear-gradient(90deg,rgba(124,58,237,0.04) 1px,transparent 1px);
  background-size:44px 44px;}
body::after{content:'';position:fixed;inset:0;pointer-events:none;z-index:0;
  background:radial-gradient(ellipse 80% 50% at 60% 0%,rgba(124,58,237,0.08) 0%,transparent 70%),
    radial-gradient(ellipse 40% 30% at 10% 80%,rgba(217,70,239,0.05) 0%,transparent 60%);}
::-webkit-scrollbar{width:3px;height:3px;}
::-webkit-scrollbar-thumb{background:var(--s4);border-radius:3px;}

@keyframes fadeUp{from{opacity:0;transform:translateY(14px);}to{opacity:1;transform:translateY(0);}}
@keyframes fadeIn{from{opacity:0;}to{opacity:1;}}
@keyframes blink{0%,100%{opacity:1;}50%{opacity:.3;}}
@keyframes pulse{0%{box-shadow:0 0 0 0 var(--dk-glow);}70%{box-shadow:0 0 0 8px transparent;}100%{box-shadow:0 0 0 0 transparent;}}
@keyframes shimmer{0%{background-position:200% 0;}100%{background-position:-200% 0;}}
@keyframes spin{from{transform:rotate(0deg);}to{transform:rotate(360deg);}}
@keyframes borderAnim{0%,100%{opacity:.4;}50%{opacity:1;}}
@keyframes msgIn{from{opacity:0;transform:translateY(8px);}to{opacity:1;transform:translateY(0);}}
@keyframes countIn{from{opacity:0;transform:translateY(6px);}to{opacity:1;transform:translateY(0);}}

/* ── SETUP SCREEN ── */
.setup-screen{
  min-height:100vh;display:flex;align-items:center;justify-content:center;
  position:relative;z-index:1;padding:20px;
}
.setup-card{
  background:var(--s1);border:1px solid var(--dk-border);
  border-radius:var(--r3);padding:36px;width:520px;max-width:100%;
  animation:fadeUp .4s ease;position:relative;overflow:hidden;
}
.setup-card::before{
  content:'';position:absolute;top:0;left:0;right:0;height:2px;
  background:linear-gradient(90deg,transparent,var(--dk),var(--neon),transparent);
}

/* ── AUTH SCREEN ── */
.auth-screen{
  min-height:100vh;display:flex;align-items:center;justify-content:center;
  position:relative;z-index:1;padding:20px;
}
.auth-card{
  background:var(--s1);border:1px solid var(--dk-border);
  border-radius:var(--r3);padding:40px;width:420px;max-width:100%;
  animation:fadeUp .4s ease;position:relative;overflow:hidden;
}
.auth-card::before{
  content:'';position:absolute;top:0;left:0;right:0;height:2px;
  background:linear-gradient(90deg,transparent,var(--dk),var(--neon),transparent);
}
.auth-logo{text-align:center;margin-bottom:28px;}
.auth-dk{font-family:var(--head);font-size:36px;font-weight:800;
  background:linear-gradient(135deg,var(--dk-xl),var(--neon));
  -webkit-background-clip:text;-webkit-text-fill-color:transparent;background-clip:text;
  display:inline-block;margin-bottom:4px;}
.auth-academy{font-family:var(--head);font-size:13px;letter-spacing:3px;color:var(--t2);font-weight:600;}
.auth-by{font-size:11px;color:var(--t3);font-family:var(--mono);margin-top:4px;}
.auth-tabs{display:flex;gap:0;margin-bottom:24px;background:var(--s2);border-radius:var(--r);padding:3px;}
.auth-tab{flex:1;padding:8px;border-radius:7px;border:none;cursor:pointer;font-size:13px;font-weight:700;
  font-family:var(--font);transition:all .15s;color:var(--t3);background:transparent;}
.auth-tab.on{background:var(--dk);color:#fff;box-shadow:0 0 12px var(--dk-glow);}
.auth-divider{display:flex;align-items:center;gap:10px;margin:16px 0;}
.auth-divider-line{flex:1;height:1px;background:var(--border);}
.auth-divider-txt{font-size:11px;color:var(--t3);font-family:var(--mono);}
.auth-error{
  background:var(--dn-dim);border:1px solid rgba(244,63,94,.2);
  border-radius:var(--r);padding:10px 14px;font-size:12.5px;color:var(--dn);margin-bottom:14px;
}
.auth-success{
  background:var(--up-dim);border:1px solid rgba(34,197,94,.2);
  border-radius:var(--r);padding:10px 14px;font-size:12.5px;color:var(--up);margin-bottom:14px;
}

/* ── APP LAYOUT ── */
.app{display:flex;height:100vh;overflow:hidden;position:relative;z-index:1;}
.sb{
  width:230px;flex-shrink:0;background:var(--s1);border-right:1px solid var(--border);
  display:flex;flex-direction:column;position:relative;z-index:20;overflow:hidden;
}
.sb::after{
  content:'';position:absolute;top:0;right:0;bottom:0;width:1px;
  background:linear-gradient(180deg,transparent 0%,var(--dk) 30%,var(--neon) 70%,transparent 100%);
  animation:borderAnim 3s ease infinite;
}
.sb-brand{padding:18px 16px 14px;border-bottom:1px solid var(--border);position:relative;}
.sb-brand::before{content:'';position:absolute;top:0;left:0;right:0;height:1px;
  background:linear-gradient(90deg,transparent,var(--dk),var(--neon),transparent);}
.brand-row{display:flex;align-items:center;gap:10px;}
.brand-icon{
  width:40px;height:40px;border-radius:10px;flex-shrink:0;
  background:linear-gradient(135deg,var(--dk),var(--neon));
  display:flex;align-items:center;justify-content:center;
  font-family:var(--head);font-size:14px;font-weight:800;color:#fff;
  box-shadow:0 0 20px var(--dk-glow);}
.brand-name{font-family:var(--head);font-size:12px;font-weight:700;letter-spacing:1px;
  background:linear-gradient(90deg,var(--dk-xl),var(--neon));
  -webkit-background-clip:text;-webkit-text-fill-color:transparent;background-clip:text;}
.brand-by{font-size:10px;color:var(--t3);font-family:var(--mono);letter-spacing:.5px;}
.brand-live{display:flex;align-items:center;gap:6px;margin-top:10px;padding:5px 10px;
  background:var(--s2);border:1px solid var(--border);border-radius:20px;}
.live-dot{width:6px;height:6px;border-radius:50%;background:var(--up);
  box-shadow:0 0 6px var(--up);animation:blink 2s ease infinite;}
.live-txt{font-size:10px;font-family:var(--mono);color:var(--t3);letter-spacing:.5px;}
.live-val{margin-left:auto;font-size:11px;font-family:var(--mono);color:var(--dk-xl);font-weight:600;}
.sb-nav{flex:1;overflow-y:auto;padding:12px 10px;}
.sb-sec{font-size:9px;font-weight:600;letter-spacing:2px;text-transform:uppercase;
  color:var(--t4);padding:10px 8px 5px;font-family:var(--mono);}
.sb-item{
  display:flex;align-items:center;gap:9px;padding:9px 10px;border-radius:var(--r);
  cursor:pointer;font-size:13px;font-weight:600;color:var(--t3);
  transition:all .15s;position:relative;user-select:none;margin-bottom:1px;
}
.sb-item:hover{background:var(--s2);color:var(--t2);}
.sb-item.on{background:var(--dk-dim);color:var(--dk-xl);}
.sb-item.on::before{
  content:'';position:absolute;left:0;top:20%;bottom:20%;width:2px;
  border-radius:0 2px 2px 0;background:linear-gradient(180deg,var(--dk-xl),var(--neon));
  box-shadow:0 0 8px var(--dk-glow);}
.sb-ico{font-size:13px;width:17px;text-align:center;flex-shrink:0;}
.sb-badge{
  margin-left:auto;font-size:9.5px;font-weight:700;font-family:var(--mono);
  padding:1px 7px;border-radius:20px;min-width:20px;text-align:center;
}
.sb-badge.purple{background:var(--dk-dim);color:var(--dk-xl);border:1px solid var(--dk-border);}
.sb-badge.green{background:var(--up-dim);color:var(--up);}
.sb-ai{margin:10px;padding:12px 14px;border-radius:var(--r2);
  background:linear-gradient(135deg,var(--dk-dim),var(--neon-dim));
  border:1px solid var(--dk-border);cursor:pointer;transition:all .2s;}
.sb-ai:hover{border-color:var(--dk);box-shadow:0 0 20px rgba(124,58,237,.15);}
.sb-ai-title{font-size:11px;font-weight:700;color:var(--dk-xl);margin-bottom:3px;font-family:var(--head);letter-spacing:.5px;}
.sb-ai-sub{font-size:10.5px;color:var(--t3);line-height:1.4;}
.sb-foot{padding:12px 14px;border-top:1px solid var(--border);display:flex;align-items:center;gap:9px;}
.sb-avatar{
  width:32px;height:32px;border-radius:50%;flex-shrink:0;
  background:linear-gradient(135deg,var(--dk),var(--neon));
  display:flex;align-items:center;justify-content:center;
  font-family:var(--head);font-size:10px;font-weight:700;color:#fff;
  box-shadow:0 0 12px var(--dk-glow);}
.sb-uname{font-size:12px;font-weight:700;color:var(--t1);}
.sb-uplan{font-size:9.5px;color:var(--dk-xl);font-family:var(--mono);letter-spacing:.5px;}
.sb-logout{
  margin-left:auto;background:none;border:none;color:var(--t3);cursor:pointer;
  font-size:14px;padding:4px;border-radius:4px;transition:color .15s;
}
.sb-logout:hover{color:var(--dn);}

/* ── MAIN ── */
.main{flex:1;overflow-y:auto;display:flex;flex-direction:column;}
.topbar{
  display:flex;align-items:flex-start;justify-content:space-between;
  padding:18px 28px 16px;border-bottom:1px solid var(--border);
  position:sticky;top:0;z-index:10;
  background:rgba(7,5,14,.9);backdrop-filter:blur(20px);
}
.topbar::after{
  content:'';position:absolute;bottom:0;left:0;right:0;height:1px;
  background:linear-gradient(90deg,transparent,var(--dk-border),transparent);}
.pg-title{font-family:var(--head);font-size:17px;font-weight:700;letter-spacing:1px;color:var(--t1);}
.pg-title em{font-style:normal;color:var(--dk-xl);}
.pg-sub{font-size:11px;color:var(--t3);font-family:var(--mono);margin-top:3px;letter-spacing:.3px;}
.topbar-r{display:flex;align-items:center;gap:8px;margin-top:2px;}
.content{padding:20px 28px;display:flex;flex-direction:column;gap:16px;}

/* ── BUTTONS ── */
.btn{display:inline-flex;align-items:center;gap:6px;border:none;border-radius:var(--r);
  padding:9px 16px;font-size:12.5px;font-weight:600;cursor:pointer;font-family:var(--font);
  transition:all .15s;letter-spacing:.3px;}
.btn-purple{background:linear-gradient(135deg,var(--dk),var(--neon));color:#fff;}
.btn-purple:hover{box-shadow:0 0 20px var(--dk-glow);transform:translateY(-1px);}
.btn-ghost{background:var(--s2);border:1px solid var(--border2);color:var(--t2);}
.btn-ghost:hover{background:var(--s3);color:var(--t1);}
.btn-red{background:var(--dn);color:#fff;}
.btn-red:hover{background:#fb7185;}
.btn-sm{padding:7px 13px;font-size:12px;}
.btn-xs{padding:4px 9px;font-size:11px;}
.btn:disabled{opacity:.5;cursor:not-allowed;transform:none !important;box-shadow:none !important;}

/* ── INPUTS ── */
.inp{width:100%;background:var(--s2);border:1px solid var(--border2);
  border-radius:var(--r);padding:9px 12px;font-size:13px;color:var(--t1);font-family:var(--font);
  outline:none;transition:border-color .15s;}
.inp:focus{border-color:var(--dk);box-shadow:0 0 0 3px rgba(124,58,237,.1);}
.inp::placeholder{color:var(--t3);}
.inp option{background:var(--s3);}
.lbl{font-size:10.5px;font-weight:600;color:var(--t3);margin-bottom:5px;letter-spacing:.5px;text-transform:uppercase;font-family:var(--mono);}

/* ── CARD ── */
.card{background:var(--s1);border:1px solid var(--border);border-radius:var(--r2);padding:20px;
  position:relative;overflow:hidden;animation:fadeUp .3s ease;}
.card::before{content:'';position:absolute;top:0;left:20%;right:20%;height:1px;
  background:linear-gradient(90deg,transparent,var(--dk-border),transparent);}
.card-title{font-size:10px;font-weight:600;letter-spacing:1.8px;text-transform:uppercase;
  color:var(--t3);margin-bottom:14px;font-family:var(--mono);}

/* ── KPI ── */
.kpi-strip{display:grid;grid-template-columns:repeat(4,1fr);gap:12px;}
.kpi{background:var(--s1);border:1px solid var(--border);border-radius:var(--r2);
  padding:18px 20px;position:relative;overflow:hidden;transition:all .2s;cursor:default;}
.kpi:hover{transform:translateY(-2px);border-color:var(--dk-border);}
.kpi-stripe{position:absolute;top:0;left:0;right:0;height:2px;}
.kpi-ico{width:34px;height:34px;border-radius:8px;display:flex;align-items:center;
  justify-content:center;font-size:15px;margin-bottom:10px;}
.kpi-lbl{font-size:10.5px;font-weight:600;color:var(--t3);letter-spacing:.3px;margin-bottom:5px;font-family:var(--mono);}
.kpi-val{font-size:26px;font-weight:700;font-family:var(--mono);letter-spacing:-1.5px;line-height:1;animation:countIn .4s ease;}
.kpi-sub{font-size:10.5px;color:var(--t3);margin-top:5px;font-family:var(--mono);}
.kpi-delta{position:absolute;top:16px;right:14px;font-size:10px;font-weight:700;
  padding:2px 8px;border-radius:20px;font-family:var(--mono);}
.kpi-delta.up{background:var(--up-dim);color:var(--up);}
.kpi-delta.dn{background:var(--dn-dim);color:var(--dn);}

/* ── INSIGHTS ── */
.ins-row{display:grid;grid-template-columns:repeat(3,1fr);gap:10px;}
.ins{background:var(--s1);border:1px solid var(--border);border-radius:var(--r2);padding:16px;
  transition:border-color .2s;animation:fadeUp .35s ease both;position:relative;overflow:hidden;}
.ins::after{content:'';position:absolute;bottom:0;left:0;right:0;height:1px;
  background:linear-gradient(90deg,transparent,var(--dk-border),transparent);}
.ins:hover{border-color:var(--dk-border);}
.ins:nth-child(2){animation-delay:.08s;}.ins:nth-child(3){animation-delay:.16s;}
.ins-ico{font-size:22px;margin-bottom:9px;}
.ins-txt{font-size:12.5px;line-height:1.6;color:var(--t2);}
.ins-tag{display:inline-flex;align-items:center;gap:4px;font-size:10px;font-weight:700;
  padding:2px 9px;border-radius:20px;margin-top:9px;letter-spacing:.5px;text-transform:uppercase;font-family:var(--mono);}
.ins-tag.opp{background:var(--up-dim);color:var(--up);}
.ins-tag.warn{background:var(--warn-dim);color:var(--warn);}
.ins-tag.info{background:var(--dk-dim);color:var(--dk-xl);}
.ins-tag.alert{background:var(--dn-dim);color:var(--dn);}
.ins-skel{background:linear-gradient(90deg,var(--s2) 25%,var(--s3) 50%,var(--s2) 75%);
  background-size:200% 100%;animation:shimmer 1.5s infinite;border-radius:4px;}

/* ── SCORE ── */
.score-ring{position:relative;display:inline-flex;align-items:center;justify-content:center;}
.score-inner{position:absolute;text-align:center;font-family:var(--mono);font-weight:700;}

/* ── PIPELINE ── */
.pipe-wrap{overflow-x:auto;padding-bottom:6px;}
.pipeline{display:flex;gap:10px;min-width:max-content;}
.pcol{width:234px;flex-shrink:0;}
.pcol-hd{display:flex;align-items:center;gap:7px;padding:10px 12px;background:var(--s2);
  border:1px solid var(--border);border-bottom:none;border-radius:var(--r) var(--r) 0 0;}
.pcol-dot{width:7px;height:7px;border-radius:50%;flex-shrink:0;}
.pcol-name{font-size:11.5px;font-weight:700;flex:1;font-family:var(--mono);}
.pcol-cnt{font-size:9.5px;font-family:var(--mono);background:var(--s3);padding:1px 7px;border-radius:20px;color:var(--t3);}
.pcol-body{background:var(--s1);border:1px solid var(--border);border-top:none;
  border-radius:0 0 var(--r) var(--r);min-height:280px;padding:7px;display:flex;flex-direction:column;gap:6px;}
.deal{background:var(--s2);border:1px solid var(--border2);border-radius:var(--r);
  padding:12px;cursor:pointer;transition:all .15s;position:relative;}
.deal:hover{border-color:var(--dk-border);background:var(--s3);}
.deal-hnd{font-size:12.5px;font-weight:700;font-family:var(--mono);}
.deal-ni{font-size:11px;color:var(--t3);margin:2px 0 7px;}
.deal-val{font-size:15px;font-weight:700;font-family:var(--mono);color:var(--dk-xl);}
.deal-cli{font-size:10.5px;color:var(--t3);margin-top:3px;}
.deal-age{position:absolute;top:9px;right:9px;font-size:9.5px;font-family:var(--mono);padding:2px 6px;border-radius:20px;}
.deal-age.f{background:var(--up-dim);color:var(--up);}
.deal-age.m{background:var(--warn-dim);color:var(--warn);}
.deal-age.o{background:var(--dn-dim);color:var(--dn);}
.deal-nota{font-size:10.5px;color:var(--t3);margin-top:5px;line-height:1.4;}
.deal-btns{display:flex;gap:4px;margin-top:8px;}
.pcol-total{padding:7px 12px;text-align:center;font-size:11px;font-family:var(--mono);color:var(--t3);border-top:1px solid var(--border2);}

/* ── ACCOUNTS ── */
.acct-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:10px;}
.acct{background:var(--s1);border:1px solid var(--border);border-radius:var(--r2);
  padding:16px;transition:all .2s;cursor:pointer;position:relative;overflow:hidden;}
.acct:hover{border-color:var(--dk-border);transform:translateY(-2px);box-shadow:0 8px 24px rgba(124,58,237,.12);}
.acct::before{content:'';position:absolute;top:0;left:0;right:0;height:1px;
  background:linear-gradient(90deg,transparent,var(--dk-border),transparent);}
.acct-hnd{font-size:13.5px;font-weight:700;font-family:var(--mono);}
.acct-ni{font-size:10.5px;font-weight:700;margin-top:4px;padding:2px 9px;border-radius:20px;
  display:inline-block;background:var(--dk-dim);color:var(--dk-xl);border:1px solid var(--dk-border);}
.acct-stats{display:grid;grid-template-columns:repeat(3,1fr);gap:6px;margin-top:12px;}
.acct-sl{font-size:10px;color:var(--t3);margin-bottom:2px;font-family:var(--mono);}
.acct-sv{font-size:13px;font-weight:700;font-family:var(--mono);}
.acct-bot{display:flex;align-items:center;justify-content:space-between;margin-top:10px;}
.avail{font-size:10.5px;font-weight:700;padding:3px 10px;border-radius:20px;}
.avail.y{background:var(--up-dim);color:var(--up);}
.avail.n{background:var(--s3);color:var(--t3);}

/* ── CLIENTS ── */
.cli-layout{display:grid;grid-template-columns:1fr 360px;gap:14px;}
.cli-row{display:flex;align-items:center;justify-content:space-between;
  padding:12px 14px;border-radius:var(--r);cursor:pointer;
  transition:all .15s;border:1px solid transparent;margin-bottom:3px;}
.cli-row:hover{background:var(--s2);border-color:var(--border);}
.cli-row.sel{background:var(--dk-dim);border-color:var(--dk-border);}
.cli-name{font-size:14px;font-weight:700;}
.cli-meta{font-size:11px;color:var(--t3);font-family:var(--mono);margin-top:2px;}
.cli-ltv{height:3px;background:var(--s3);border-radius:3px;margin-top:5px;overflow:hidden;}
.cli-ltv-fill{height:100%;border-radius:3px;background:linear-gradient(90deg,var(--dk),var(--neon));}
.cli-total{font-family:var(--mono);font-weight:700;font-size:13px;color:var(--dk-xl);}
.hist-item{display:flex;align-items:center;justify-content:space-between;
  padding:10px 12px;border-radius:var(--r);background:var(--s2);border-left:2px solid var(--dk);margin-bottom:7px;}
.hist-cnt{font-size:12px;font-weight:700;font-family:var(--mono);}
.hist-dt{font-size:10px;color:var(--t3);font-family:var(--mono);}
.hist-v{font-family:var(--mono);color:var(--dk-xl);font-size:13px;font-weight:700;}

/* ── CHAT ── */
.chat-page{display:flex;flex-direction:column;height:calc(100vh - 72px);}
.chat-hist{flex:1;overflow-y:auto;padding:18px 28px;display:flex;flex-direction:column;gap:12px;}
.msg{display:flex;gap:10px;animation:msgIn .22s ease;}
.msg.u{flex-direction:row-reverse;}
.msg-av{width:32px;height:32px;border-radius:50%;flex-shrink:0;
  display:flex;align-items:center;justify-content:center;font-size:10px;font-weight:700;}
.msg-av.ai{background:linear-gradient(135deg,var(--dk),var(--neon));color:#fff;font-family:var(--head);}
.msg-av.u{background:var(--s3);color:var(--dk-xl);font-family:var(--head);}
.msg-bub{max-width:70%;padding:12px 16px;border-radius:var(--r2);font-size:13.5px;line-height:1.65;}
.msg-bub.ai{background:var(--s2);border:1px solid var(--border);border-radius:6px var(--r2) var(--r2) var(--r2);}
.msg-bub.ai strong{color:var(--dk-xl);}
.msg-bub.u{background:var(--dk-dim);border:1px solid var(--dk-border);border-radius:var(--r2) 6px var(--r2) var(--r2);}
.typing{display:flex;align-items:center;gap:5px;padding:4px 2px;}
.typing span{width:7px;height:7px;border-radius:50%;background:var(--dk-xl);animation:blink 1.2s ease infinite;}
.typing span:nth-child(2){animation-delay:.2s;}.typing span:nth-child(3){animation-delay:.4s;}
.sugs{display:flex;flex-wrap:wrap;gap:6px;padding:8px 28px 4px;}
.chip{background:var(--s2);border:1px solid var(--border2);border-radius:20px;
  padding:5px 13px;font-size:11.5px;font-weight:500;color:var(--t3);
  cursor:pointer;transition:all .15s;font-family:var(--font);}
.chip:hover{border-color:var(--dk-border);color:var(--dk-xl);background:var(--dk-dim2);}
.chat-bar{padding:14px 28px 18px;border-top:1px solid var(--border);
  display:flex;gap:10px;align-items:flex-end;background:rgba(7,5,14,.85);backdrop-filter:blur(12px);}
.chat-inp{flex:1;background:var(--s2);border:1px solid var(--border2);border-radius:var(--r2);
  padding:11px 16px;font-size:13.5px;color:var(--t1);font-family:var(--font);
  outline:none;transition:border-color .15s;resize:none;min-height:44px;max-height:120px;}
.chat-inp:focus{border-color:var(--dk);box-shadow:0 0 0 3px rgba(124,58,237,.1);}
.chat-inp::placeholder{color:var(--t3);}
.send-btn{width:44px;height:44px;border-radius:50%;flex-shrink:0;border:none;
  background:linear-gradient(135deg,var(--dk),var(--neon));color:#fff;cursor:pointer;font-size:16px;
  display:flex;align-items:center;justify-content:center;transition:all .15s;}
.send-btn:hover{box-shadow:0 0 20px var(--dk-glow);transform:scale(1.08);}
.send-btn:disabled{background:var(--s3);cursor:not-allowed;transform:none;box-shadow:none;}

/* ── LOADING ── */
.loading-screen{min-height:100vh;display:flex;align-items:center;justify-content:center;
  flex-direction:column;gap:14px;position:relative;z-index:1;}
.spinner{width:40px;height:40px;border:3px solid var(--dk-dim);border-top-color:var(--dk);
  border-radius:50%;animation:spin .8s linear infinite;}

/* ── GOAL ── */
.goal-track{height:8px;background:var(--s3);border-radius:20px;overflow:hidden;margin:8px 0;}
.goal-fill{height:100%;border-radius:20px;background:linear-gradient(90deg,var(--dk),var(--neon));
  box-shadow:0 0 12px var(--dk-glow);transition:width 1.2s cubic-bezier(.4,0,.2,1);}

/* ── MODAL ── */
.overlay{position:fixed;inset:0;background:rgba(0,0,0,.82);display:flex;align-items:center;
  justify-content:center;z-index:200;backdrop-filter:blur(8px);animation:fadeIn .15s;}
.modal{background:var(--s1);border:1px solid var(--dk-border);border-radius:var(--r3);
  padding:28px;width:480px;max-width:92vw;animation:fadeUp .2s;position:relative;overflow:hidden;}
.modal::before{content:'';position:absolute;top:0;left:0;right:0;height:1px;
  background:linear-gradient(90deg,transparent,var(--dk),var(--neon),transparent);}
.modal-title{font-family:var(--head);font-size:15px;font-weight:700;margin-bottom:20px;letter-spacing:.5px;}
.modal-title em{font-style:normal;color:var(--dk-xl);}
.form2{display:grid;grid-template-columns:1fr 1fr;gap:10px;}
.form-row{margin-bottom:11px;}
.form-acts{display:flex;gap:8px;justify-content:flex-end;margin-top:18px;}

/* ── TABLE ── */
table{width:100%;border-collapse:collapse;}
thead th{font-size:9.5px;font-weight:600;letter-spacing:1.5px;text-transform:uppercase;
  color:var(--t3);padding:9px 14px;text-align:left;border-bottom:1px solid var(--border);font-family:var(--mono);}
tbody tr{border-bottom:1px solid rgba(255,255,255,0.03);transition:background .12s;cursor:pointer;}
tbody tr:hover{background:var(--s2);}
tbody td{padding:11px 14px;font-size:13px;}
td.mo{font-family:var(--mono);font-size:12px;}
td.pu{color:var(--dk-xl);font-family:var(--mono);font-weight:700;}
.pill{display:inline-flex;padding:2px 9px;border-radius:20px;font-size:10.5px;font-weight:700;}
.pill.pu{background:var(--dk-dim);color:var(--dk-xl);border:1px solid var(--dk-border);}
.pill.gr{background:var(--up-dim);color:var(--up);}
.pill.am{background:var(--warn-dim);color:var(--warn);}

/* ── ANALYTICS ── */
.anal-2{display:grid;grid-template-columns:3fr 2fr;gap:12px;}
.anal-3{display:grid;grid-template-columns:repeat(3,1fr);gap:12px;}
.mini-s{background:var(--s1);border:1px solid var(--border);border-radius:var(--r2);padding:16px;}
.mini-s-l{font-size:10.5px;color:var(--t3);font-family:var(--mono);margin-bottom:5px;letter-spacing:.5px;}
.mini-s-v{font-size:22px;font-weight:700;font-family:var(--mono);letter-spacing:-.5px;}
.mini-s-bar{margin-top:8px;height:4px;border-radius:4px;background:var(--s3);overflow:hidden;}
.mini-s-fill{height:100%;border-radius:4px;}

.ct{background:var(--s2);border:1px solid var(--dk-border);border-radius:8px;padding:10px 14px;font-family:var(--mono);font-size:12px;}
.ct-l{color:var(--t3);font-size:10px;margin-bottom:3px;}
.ct-v{color:var(--dk-xl);font-weight:700;}

/* ── EMPTY STATE ── */
.empty{display:flex;flex-direction:column;align-items:center;justify-content:center;
  padding:48px;color:var(--t3);text-align:center;gap:10px;}
.empty-ico{font-size:36px;opacity:.4;}
.empty-txt{font-size:13px;line-height:1.5;}
`}</style>;

/* ══════════ UTILS ══════════════════════════════════════════════ */
const fmt  = v => `R$ ${Number(v||0).toLocaleString("pt-BR",{minimumFractionDigits:2})}`;
const fmtS = v => (v||0)>=1000000?`R$${(v/1e6).toFixed(1)}M`:(v||0)>=1000?`R$${(v/1000).toFixed(1)}K`:`R$${v||0}`;
const todayStr = () => new Date().toLocaleDateString("pt-BR");

const NICHOS=["Mulher da Roça","Homem da Roça","Fitness","Culinária","Moda","Humor","Pets","Negócios","Lifestyle","Beleza"];
const STAGES=[
  {id:"lead",label:"Lead Novo",color:"#A78BFA"},
  {id:"contato",label:"Contato",color:"#38BDF8"},
  {id:"negociacao",label:"Negociação",color:"#F59E0B"},
  {id:"proposta",label:"Proposta",color:"#C084FC"},
  {id:"fechado",label:"✓ Fechado",color:"#22C55E"},
];
const PU=["#7C3AED","#9D5FF5","#C084FC","#D946EF","#A78BFA"];

function ScoreRing({score,size=54}){
  const r=(size-10)/2,c=2*Math.PI*r,d=((score||0)/100)*c;
  const col=score>=80?"#22C55E":score>=55?"#F59E0B":"#F43F5E";
  const grade=score>=80?"A":score>=65?"B":score>=50?"C":"D";
  return(
    <div className="score-ring" style={{width:size,height:size}}>
      <svg width={size} height={size} style={{transform:"rotate(-90deg)"}}>
        <circle cx={size/2} cy={size/2} r={r} fill="none" stroke="rgba(124,58,237,0.15)" strokeWidth={6}/>
        <circle cx={size/2} cy={size/2} r={r} fill="none" stroke={col} strokeWidth={6}
          strokeDasharray={`${d} ${c-d}`} strokeLinecap="round"
          style={{transition:"stroke-dasharray .8s cubic-bezier(.4,0,.2,1)"}}/>
      </svg>
      <div className="score-inner" style={{color:col}}>
        <div style={{fontSize:size<50?11:14,lineHeight:1,fontWeight:700}}>{grade}</div>
        <div style={{fontSize:9,color:"var(--t3)",marginTop:1}}>{score}</div>
      </div>
    </div>
  );
}
function computeScore(a){
  const f=parseFloat(a.seguidores||"0")*(String(a.seguidores||"").toUpperCase().includes("K")?1000:1);
  return Math.min(Math.round(
    Math.min((f/120000)*40,40)+Math.min((Number(a.videos)||0)/400*28,28)+
    (a.status==="available"?14:4)+(["Moda","Humor","Fitness"].includes(a.nicho)?18:8)
  ),100);
}
const CT=({active,payload,label})=>active&&payload?.length?(
  <div className="ct"><div className="ct-l">{label}</div><div className="ct-v">{fmtS(payload[0]?.value||0)}</div></div>
):null;

/* ══════════ SETUP SCREEN ═══════════════════════════════════════ */
function SetupScreen({onSave}){
  const [url,setUrl]=useState("");
  const [key,setKey]=useState("");
  return(
    <div className="setup-screen">
      <div className="setup-card">
        <div style={{textAlign:"center",marginBottom:28}}>
          <div style={{fontFamily:"var(--head)",fontSize:32,fontWeight:800,
            background:"linear-gradient(135deg,var(--dk-xl),var(--neon))",
            WebkitBackgroundClip:"text",WebkitTextFillColor:"transparent",backgroundClip:"text",marginBottom:6}}>
            DK ACADEMY
          </div>
          <div style={{fontSize:11,color:"var(--t3)",fontFamily:"var(--mono)"}}>by Raveli DK · Configuração Inicial</div>
        </div>
        <div style={{background:"var(--s2)",border:"1px solid var(--border)",borderRadius:var(--r2),padding:"14px 16px",marginBottom:20}}>
          <div style={{fontSize:12,color:"var(--dk-xl)",fontWeight:700,marginBottom:8,fontFamily:"var(--mono)"}}>📋 PASSO A PASSO</div>
          {["1. Acesse supabase.com e crie um projeto gratuito",
            "2. Vá em SQL Editor > New Query",
            "3. Cole e execute o arquivo dk-academy-supabase.sql",
            "4. Vá em Project Settings > API",
            "5. Copie a URL e a anon key abaixo"].map((t,i)=>(
            <div key={i} style={{fontSize:12,color:"var(--t2)",marginBottom:4,display:"flex",gap:8}}>
              <span>{t}</span>
            </div>
          ))}
        </div>
        <div className="form-row"><div className="lbl">Supabase Project URL</div>
          <input className="inp" placeholder="https://xxxxxxxxxxxx.supabase.co" value={url} onChange={e=>setUrl(e.target.value)}/>
        </div>
        <div className="form-row"><div className="lbl">Anon Public Key</div>
          <input className="inp" placeholder="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." value={key} onChange={e=>setKey(e.target.value)}/>
        </div>
        <button className="btn btn-purple" style={{width:"100%",marginTop:8,justifyContent:"center"}}
          onClick={()=>{if(url&&key)onSave(url,key);}}>
          ✓ Conectar ao Banco de Dados
        </button>
      </div>
    </div>
  );
}

/* ══════════ AUTH SCREEN ════════════════════════════════════════ */
function AuthScreen(){
  const [tab,setTab]=useState("login");
  const [email,setEmail]=useState("");
  const [pass,setPass]=useState("");
  const [nome,setNome]=useState("");
  const [loading,setLoading]=useState(false);
  const [error,setError]=useState("");
  const [success,setSuccess]=useState("");

  const handleLogin=async()=>{
    if(!email||!pass){setError("Preencha email e senha.");return;}
    setLoading(true);setError("");
    const d=await sb.signIn(email,pass);
    if(d.error){setError(d.error.message||"Email ou senha incorretos.");}
    setLoading(false);
  };

  const handleRegister=async()=>{
    if(!email||!pass||!nome){setError("Preencha todos os campos.");return;}
    if(pass.length<6){setError("Senha deve ter mínimo 6 caracteres.");return;}
    setLoading(true);setError("");
    const d=await sb.signUp(email,pass);
    if(d.error){setError(d.error.message||"Erro ao cadastrar.");}
    else if(d.user){
      await sb.updateProfile({nome,email});
      setSuccess("Conta criada! Pode fazer login agora.");setTab("login");
    }
    setLoading(false);
  };

  return(
    <div className="auth-screen">
      <div className="auth-card">
        <div className="auth-logo">
          <div className="auth-dk">DK</div>
          <div className="auth-academy">ACADEMY</div>
          <div className="auth-by">by Raveli DK</div>
        </div>
        <div className="auth-tabs">
          <button className={`auth-tab ${tab==="login"?"on":""}`} onClick={()=>{setTab("login");setError("");}}>Entrar</button>
          <button className={`auth-tab ${tab==="register"?"on":""}`} onClick={()=>{setTab("register");setError("");}}>Cadastrar</button>
        </div>
        {error&&<div className="auth-error">⚠ {error}</div>}
        {success&&<div className="auth-success">✓ {success}</div>}
        {tab==="register"&&(
          <div className="form-row"><div className="lbl">Seu Nome</div>
            <input className="inp" placeholder="Raveli DK" value={nome} onChange={e=>setNome(e.target.value)}/>
          </div>
        )}
        <div className="form-row"><div className="lbl">Email</div>
          <input className="inp" placeholder="seu@email.com" type="email" value={email} onChange={e=>setEmail(e.target.value)}
            onKeyDown={e=>e.key==="Enter"&&(tab==="login"?handleLogin():handleRegister())}/>
        </div>
        <div className="form-row"><div className="lbl">Senha</div>
          <input className="inp" placeholder="••••••••" type="password" value={pass} onChange={e=>setPass(e.target.value)}
            onKeyDown={e=>e.key==="Enter"&&(tab==="login"?handleLogin():handleRegister())}/>
        </div>
        <button className="btn btn-purple" style={{width:"100%",justifyContent:"center",marginTop:4}}
          disabled={loading} onClick={tab==="login"?handleLogin:handleRegister}>
          {loading?"Aguarde…":tab==="login"?"→ Entrar na DK Academy":"→ Criar Minha Conta"}
        </button>
        <p style={{fontSize:11,color:"var(--t3)",textAlign:"center",marginTop:16,lineHeight:1.5}}>
          {tab==="login"?"Não tem conta? ":"Já tem conta? "}
          <span style={{color:"var(--dk-xl)",cursor:"pointer"}} onClick={()=>{setTab(tab==="login"?"register":"login");setError("");}}>
            {tab==="login"?"Cadastre-se →":"Faça login →"}
          </span>
        </p>
      </div>
    </div>
  );
}

/* ══════════ DASHBOARD ══════════════════════════════════════════ */
function Dashboard({sales,accounts,pipeline,profile,onNewSale,onGoalUpdate}){
  const [insights,setInsights]=useState(null);
  const [loadIns,setLoadIns]=useState(true);
  const total=sales.reduce((a,s)=>a+(s.valor||0),0);
  const goal=profile?.meta_mensal||10000;
  const pct=Math.min((total/goal)*100,100);
  const avail=accounts.filter(a=>a.status==="available").length;
  const pipeV=pipeline.reduce((a,d)=>a+(d.valor||0),0);

  const chartData=sales.slice(0,8).reverse().map((s,i)=>({w:s.data?.slice(0,5)||`d${i}`,r:s.valor||0,f:(s.valor||0)*1.2}));

  useEffect(()=>{
    const ctx={total_faturado:total,contas_vendidas:sales.length,contas_disponiveis:avail,
      pipeline_deals:pipeline.length,pipeline_valor:pipeV,meta_mensal:goal};
    fetch(`${sb.url}/v1/messages` .replace("rest/v1",""),{}).catch(()=>{});
    fetch("https://api.anthropic.com/v1/messages",{
      method:"POST",headers:{"Content-Type":"application/json"},
      body:JSON.stringify({
        model:"claude-sonnet-4-20250514",max_tokens:600,
        system:`Você é o DK Advisor da DK Academy by Raveli DK. Retorne SOMENTE JSON puro sem markdown: {"insights":[{"icon":"emoji","text":"max 22 palavras","tag":"opp|warn|info|alert","tagLabel":"2 palavras"}]}. Exatamente 3 insights acionáveis.`,
        messages:[{role:"user",content:`Dados: ${JSON.stringify(ctx)}`}]
      })
    }).then(r=>r.json()).then(d=>{
      setInsights(JSON.parse((d.content?.[0]?.text||"{}").replace(/```json|```/g,"").trim()).insights||[]);
    }).catch(()=>setInsights([
      {icon:"🔥",text:`Pipeline tem ${pipeline.length} deals — foque em fechar os mais avançados essa semana.`,tag:"opp",tagLabel:"Urgente"},
      {icon:"📊",text:`Meta ${pct.toFixed(0)}% atingida de ${fmt(goal)} — ${pct>=100?"parabéns, eleve a meta!":"mantenha o ritmo!"}`,tag:"info",tagLabel:"Meta"},
      {icon:"💡",text:`${avail} contas disponíveis — crie urgência nos leads com oferta por tempo limitado.`,tag:"warn",tagLabel:"Ação"},
    ])).finally(()=>setLoadIns(false));
  },[sales.length,pipeline.length]);

  return(
    <>
      <div className="topbar">
        <div>
          <div className="pg-title">DASHBOARD — <em>VISÃO GERAL</em></div>
          <div className="pg-sub">Olá, {profile?.nome||"usuário"} · DK Academy · ao vivo</div>
        </div>
        <div className="topbar-r">
          <button className="btn btn-ghost btn-sm">↓ Exportar</button>
          <button className="btn btn-purple btn-sm" onClick={onNewSale}>+ Nova Venda</button>
        </div>
      </div>
      <div className="content">
        <div className="kpi-strip">
          {[
            {lbl:"Faturamento",val:fmtS(total),sub:fmt(total),ico:"◈",ibg:"rgba(124,58,237,.12)",stripe:"linear-gradient(90deg,var(--dk),transparent)",delta:`${sales.length} vendas`,up:true},
            {lbl:"Pipeline",val:fmtS(pipeV),sub:`${pipeline.length} deals`,ico:"⬡",ibg:"rgba(217,70,239,.1)",stripe:"linear-gradient(90deg,var(--neon),transparent)",delta:"em jogo",up:true},
            {lbl:"Disponíveis",val:avail,sub:"contas",ico:"◉",ibg:"rgba(34,197,94,.1)",stripe:"linear-gradient(90deg,var(--up),transparent)",delta:"p/ venda",up:true},
            {lbl:"Ticket Médio",val:fmtS(sales.length?Math.round(total/sales.length):0),sub:"por venda",ico:"◆",ibg:"rgba(56,189,248,.1)",stripe:"linear-gradient(90deg,var(--info),transparent)",delta:"+12%",up:true},
          ].map((k,i)=>(
            <div key={i} className="kpi" style={{animationDelay:`${i*.07}s`}}>
              <div className="kpi-stripe" style={{background:k.stripe}}/>
              <div className="kpi-ico" style={{background:k.ibg,color:"var(--dk-xl)",fontSize:16}}>{k.ico}</div>
              <div className="kpi-lbl">{k.lbl}</div>
              <div className="kpi-val">{k.val}</div>
              <div className="kpi-sub">{k.sub}</div>
              <div className={`kpi-delta ${k.up?"up":"dn"}`}>{k.delta}</div>
            </div>
          ))}
        </div>

        <div className="card">
          <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:10}}>
            <div className="card-title" style={{margin:0}}>META DO MÊS</div>
            <div style={{display:"flex",alignItems:"center",gap:10}}>
              <span style={{fontFamily:"var(--mono)",fontSize:13,color:"var(--dk-xl)",fontWeight:700}}>{fmt(total)}</span>
              <span style={{fontFamily:"var(--mono)",fontSize:12,color:"var(--t3)"}}>/ {fmt(goal)}</span>
              <span style={{background:"var(--up-dim)",color:"var(--up)",padding:"2px 8px",borderRadius:20,fontSize:11,fontWeight:700}}>{pct.toFixed(0)}%</span>
              <input className="inp" style={{width:110,fontSize:12,padding:"5px 8px"}} type="number" placeholder="Nova meta" onKeyDown={e=>{if(e.key==="Enter"){onGoalUpdate(Number(e.target.value));e.target.value="";}}}/>
            </div>
          </div>
          <div className="goal-track"><div className="goal-fill" style={{width:`${pct}%`}}/></div>
        </div>

        <div>
          <div style={{display:"flex",alignItems:"center",gap:10,marginBottom:10}}>
            <div style={{fontFamily:"var(--head)",fontSize:11,color:"var(--dk-xl)",letterSpacing:1}}>◈ DK ADVISOR — ANÁLISE IA</div>
            {loadIns&&<div style={{fontSize:11,color:"var(--t3)",fontFamily:"var(--mono)"}}>analisando…</div>}
          </div>
          <div className="ins-row">
            {loadIns?[0,1,2].map(i=>(
              <div key={i} className="ins" style={{minHeight:100}}>
                <div className="ins-skel" style={{width:28,height:20,marginBottom:10}}/>
                <div className="ins-skel" style={{width:"85%",height:12,marginBottom:6}}/>
                <div className="ins-skel" style={{width:"65%",height:12,marginBottom:10}}/>
                <div className="ins-skel" style={{width:70,height:18,borderRadius:20}}/>
              </div>
            )):insights?.map((ins,i)=>(
              <div key={i} className="ins" style={{animationDelay:`${i*.1}s`}}>
                <div className="ins-ico">{ins.icon}</div>
                <div className="ins-txt">{ins.text}</div>
                <div className={`ins-tag ${ins.tag}`}>{ins.tagLabel}</div>
              </div>
            ))}
          </div>
        </div>

        <div style={{display:"grid",gridTemplateColumns:"2fr 1fr",gap:12}}>
          <div className="card">
            <div className="card-title">HISTÓRICO DE RECEITA</div>
            {chartData.length>0?(
              <ResponsiveContainer width="100%" height={200}>
                <AreaChart data={chartData}>
                  <defs>
                    <linearGradient id="gr" x1="0" y1="0" x2="0" y2="1">
                      <stop offset="0%" stopColor="#7C3AED" stopOpacity={.35}/>
                      <stop offset="100%" stopColor="#7C3AED" stopOpacity={0}/>
                    </linearGradient>
                  </defs>
                  <CartesianGrid stroke="rgba(124,58,237,0.07)" vertical={false}/>
                  <XAxis dataKey="w" tick={{fill:"#5C5478",fontSize:10,fontFamily:"JetBrains Mono"}} axisLine={false} tickLine={false}/>
                  <YAxis tickFormatter={v=>v?fmtS(v):""} tick={{fill:"#5C5478",fontSize:9,fontFamily:"JetBrains Mono"}} axisLine={false} tickLine={false}/>
                  <Tooltip content={<CT/>}/>
                  <Area type="monotone" dataKey="r" stroke="#7C3AED" strokeWidth={2.5} fill="url(#gr)" dot={{r:3,fill:"#9D5FF5"}}/>
                </AreaChart>
              </ResponsiveContainer>
            ):<div className="empty"><div className="empty-ico">📈</div><div className="empty-txt">Nenhuma venda ainda</div></div>}
          </div>
          <div className="card">
            <div className="card-title">ÚLTIMAS VENDAS</div>
            {sales.length===0?<div className="empty"><div className="empty-ico">🛒</div><div className="empty-txt">Nenhuma venda registrada</div></div>:(
              <table>
                <thead><tr><th>Conta</th><th>Valor</th></tr></thead>
                <tbody>
                  {sales.slice(0,5).map(s=>(
                    <tr key={s.id}>
                      <td style={{fontWeight:700,fontFamily:"var(--mono)",fontSize:12}}>{s.conta}</td>
                      <td className="pu">{fmt(s.valor)}</td>
                    </tr>
                  ))}
                </tbody>
              </table>
            )}
          </div>
        </div>
      </div>
    </>
  );
}

/* ══════════ PIPELINE ═══════════════════════════════════════════ */
function PipelinePage({pipeline,loadPipe}){
  const [modal,setModal]=useState(false);
  const [saving,setSaving]=useState(false);
  const [form,setForm]=useState({conta:"",nicho:"",cliente:"",valor:"",nota:"",stage:"lead"});
  const sf=(k,v)=>setForm(p=>({...p,[k]:v}));
  const ids=STAGES.map(s=>s.id);

  const move=async(deal,dir)=>{
    const ci=ids.indexOf(deal.stage),ni=ci+dir;
    if(ni<0||ni>=ids.length) return;
    await sb.update("pipeline_deals",deal.id,{stage:ids[ni]});
    loadPipe();
  };
  const remove=async(id)=>{await sb.delete("pipeline_deals",id);loadPipe();};

  return(
    <>
      <div className="topbar">
        <div>
          <div className="pg-title">PIPELINE <em>CRM</em></div>
          <div className="pg-sub">{pipeline.length} deals · {fmt(pipeline.reduce((a,d)=>a+(d.valor||0),0))} em jogo</div>
        </div>
        <button className="btn btn-purple btn-sm" onClick={()=>setModal(true)}>+ Novo Deal</button>
      </div>
      <div className="content">
        <div className="pipe-wrap">
          <div className="pipeline">
            {STAGES.map(st=>{
              const deals=pipeline.filter(d=>d.stage===st.id);
              const stV=deals.reduce((a,d)=>a+(d.valor||0),0);
              return(
                <div key={st.id} className="pcol">
                  <div className="pcol-hd">
                    <div className="pcol-dot" style={{background:st.color,boxShadow:`0 0 6px ${st.color}`}}/>
                    <div className="pcol-name">{st.label}</div>
                    <div className="pcol-cnt">{deals.length}</div>
                  </div>
                  <div className="pcol-body">
                    {deals.map(d=>{
                      const si=ids.indexOf(d.stage);
                      const dias=d.dias||0;
                      const ac=dias<=1?"f":dias<=5?"m":"o";
                      return(
                        <div key={d.id} className="deal">
                          <div className={`deal-age ${ac}`}>{dias}d</div>
                          <div className="deal-hnd">@{d.conta}</div>
                          <div className="deal-ni">{d.nicho}</div>
                          <div className="deal-val">{fmtS(d.valor)}</div>
                          <div className="deal-cli">👤 {d.cliente}</div>
                          {d.nota&&<div className="deal-nota">{d.nota}</div>}
                          <div className="deal-btns">
                            {si>0&&<button className="btn btn-ghost btn-xs" onClick={()=>move(d,-1)}>◀</button>}
                            {si<ids.length-1&&<button className="btn btn-purple btn-xs" onClick={()=>move(d,1)}>Avançar ▶</button>}
                            <button className="btn btn-ghost btn-xs" style={{marginLeft:"auto"}} onClick={()=>remove(d.id)}>✕</button>
                          </div>
                        </div>
                      );
                    })}
                    {stV>0&&<div className="pcol-total">{fmtS(stV)}</div>}
                  </div>
                </div>
              );
            })}
          </div>
        </div>
      </div>
      {modal&&(
        <div className="overlay" onClick={()=>setModal(false)}>
          <div className="modal" onClick={e=>e.stopPropagation()}>
            <div className="modal-title">NOVO DEAL — <em>PIPELINE</em></div>
            <div className="form2">
              <div className="form-row"><div className="lbl">Handle</div><input className="inp" placeholder="@conta" value={form.conta} onChange={e=>sf("conta",e.target.value)}/></div>
              <div className="form-row"><div className="lbl">Nicho</div>
                <select className="inp" value={form.nicho} onChange={e=>sf("nicho",e.target.value)}>
                  <option value="">Selecionar...</option>{NICHOS.map(n=><option key={n}>{n}</option>)}
                </select>
              </div>
              <div className="form-row"><div className="lbl">Cliente/Lead</div><input className="inp" placeholder="ID ou nome" value={form.cliente} onChange={e=>sf("cliente",e.target.value)}/></div>
              <div className="form-row"><div className="lbl">Valor (R$)</div><input className="inp" type="number" value={form.valor} onChange={e=>sf("valor",e.target.value)}/></div>
            </div>
            <div className="form-row"><div className="lbl">Estágio</div>
              <select className="inp" value={form.stage} onChange={e=>sf("stage",e.target.value)}>
                {STAGES.map(s=><option key={s.id} value={s.id}>{s.label}</option>)}
              </select>
            </div>
            <div className="form-row"><div className="lbl">Nota</div><input className="inp" placeholder="Contexto..." value={form.nota} onChange={e=>sf("nota",e.target.value)}/></div>
            <div className="form-acts">
              <button className="btn btn-ghost" onClick={()=>setModal(false)}>Cancelar</button>
              <button className="btn btn-purple" disabled={saving} onClick={async()=>{
                if(!form.conta) return;
                setSaving(true);
                await sb.insert("pipeline_deals",{...form,valor:Number(form.valor),dias:0});
                await loadPipe();
                setModal(false);setSaving(false);
                setForm({conta:"",nicho:"",cliente:"",valor:"",nota:"",stage:"lead"});
              }}>✓ Adicionar</button>
            </div>
          </div>
        </div>
      )}
    </>
  );
}

/* ══════════ CONTAS ═════════════════════════════════════════════ */
function ContasPage({accounts,loadAccounts}){
  const [search,setSearch]=useState("");
  const [filter,setFilter]=useState("all");
  const [modal,setModal]=useState(false);
  const [saving,setSaving]=useState(false);
  const [form,setForm]=useState({handle:"",nicho:"",seguidores:"",videos:"",valor:""});
  const sf=(k,v)=>setForm(p=>({...p,[k]:v}));
  const filtered=accounts.filter(a=>{
    const ms=(a.handle||"").toLowerCase().includes(search.toLowerCase())||(a.nicho||"").toLowerCase().includes(search.toLowerCase());
    const mf=filter==="all"||(filter==="avail"&&a.status==="available")||(filter==="sold"&&a.status==="sold");
    return ms&&mf;
  });
  return(
    <>
      <div className="topbar">
        <div>
          <div className="pg-title">PORTFÓLIO — <em>CONTAS</em></div>
          <div className="pg-sub">{accounts.filter(a=>a.status==="available").length} disponíveis · {accounts.filter(a=>a.status==="sold").length} vendidas</div>
        </div>
        <div className="topbar-r">
          {[["all","Todas"],["avail","Disponíveis"],["sold","Vendidas"]].map(([v,l])=>(
            <button key={v} className={`btn btn-sm ${filter===v?"btn-purple":"btn-ghost"}`} onClick={()=>setFilter(v)}>{l}</button>
          ))}
          <input className="inp" style={{width:180}} placeholder="🔍 Buscar..." value={search} onChange={e=>setSearch(e.target.value)}/>
          <button className="btn btn-purple btn-sm" onClick={()=>setModal(true)}>+ Add</button>
        </div>
      </div>
      <div className="content">
        {filtered.length===0?<div className="empty" style={{marginTop:40}}><div className="empty-ico">📱</div><div className="empty-txt">Nenhuma conta encontrada.<br/>Adicione sua primeira conta!</div></div>:(
          <div className="acct-grid">
            {filtered.map((a,i)=>{
              const sc=computeScore(a);
              return(
                <div key={a.id} className="acct" style={{animationDelay:`${i*.05}s`}}>
                  <div style={{display:"flex",alignItems:"flex-start",justifyContent:"space-between"}}>
                    <div><div className="acct-hnd">@{a.handle}</div><div className="acct-ni">{a.nicho}</div></div>
                    <ScoreRing score={sc} size={54}/>
                  </div>
                  <div className="acct-stats">
                    <div><div className="acct-sl">Seguidores</div><div className="acct-sv" style={{color:"var(--info)"}}>{a.seguidores}</div></div>
                    <div><div className="acct-sl">Vídeos</div><div className="acct-sv">{a.videos}</div></div>
                    <div><div className="acct-sl">Preço</div><div className="acct-sv" style={{color:"var(--dk-xl)"}}>{fmtS(a.valor)}</div></div>
                  </div>
                  <div className="acct-bot">
                    <span className={`avail ${a.status==="available"?"y":"n"}`}>{a.status==="available"?"✓ Disponível":"Vendida"}</span>
                    <span style={{fontSize:10,color:"var(--t3)",fontFamily:"var(--mono)"}}>{sc>=80?"Alto valor":sc>=55?"Médio":"Baixo"}</span>
                  </div>
                </div>
              );
            })}
          </div>
        )}
      </div>
      {modal&&(
        <div className="overlay" onClick={()=>setModal(false)}>
          <div className="modal" onClick={e=>e.stopPropagation()}>
            <div className="modal-title">NOVA <em>CONTA</em></div>
            <div className="form2">
              <div className="form-row"><div className="lbl">Handle</div><input className="inp" placeholder="@conta" value={form.handle} onChange={e=>sf("handle",e.target.value)}/></div>
              <div className="form-row"><div className="lbl">Nicho</div>
                <select className="inp" value={form.nicho} onChange={e=>sf("nicho",e.target.value)}>
                  <option value="">Selecionar...</option>{NICHOS.map(n=><option key={n}>{n}</option>)}
                </select>
              </div>
              <div className="form-row"><div className="lbl">Seguidores</div><input className="inp" placeholder="42.3K" value={form.seguidores} onChange={e=>sf("seguidores",e.target.value)}/></div>
              <div className="form-row"><div className="lbl">Vídeos</div><input className="inp" type="number" placeholder="0" value={form.videos} onChange={e=>sf("videos",e.target.value)}/></div>
            </div>
            <div className="form-row"><div className="lbl">Preço (R$)</div><input className="inp" type="number" placeholder="0" value={form.valor} onChange={e=>sf("valor",e.target.value)}/></div>
            <div className="form-acts">
              <button className="btn btn-ghost" onClick={()=>setModal(false)}>Cancelar</button>
              <button className="btn btn-purple" disabled={saving} onClick={async()=>{
                if(!form.handle) return;
                setSaving(true);
                await sb.insert("accounts",{...form,videos:Number(form.videos),valor:Number(form.valor),status:"available"});
                await loadAccounts();
                setModal(false);setSaving(false);
                setForm({handle:"",nicho:"",seguidores:"",videos:"",valor:""});
              }}>✓ Salvar</button>
            </div>
          </div>
        </div>
      )}
    </>
  );
}

/* ══════════ CLIENTES ═══════════════════════════════════════════ */
function ClientesPage({clients,sales,loadClients}){
  const [sel,setSel]=useState(null);
  const [nome,setNome]=useState("");
  const [contato,setContato]=useState("");
  const [notas,setNotas]=useState("");
  const [addM,setAddM]=useState(false);
  const [newN,setNewN]=useState("");
  const maxLTV=Math.max(...clients.map(c=>c.ltv||0),1);
  const pick=c=>{setSel(c);setNome(c.nome);setContato(c.contato||"");setNotas(c.notas||"");};
  const cSales=sel?sales.filter(s=>s.cliente===sel.nome):[];
  return(
    <>
      <div className="topbar">
        <div>
          <div className="pg-title">GESTÃO — <em>CLIENTES</em></div>
          <div className="pg-sub">{clients.length} clientes · LTV total {fmtS(clients.reduce((a,c)=>a+(c.ltv||0),0))}</div>
        </div>
        <button className="btn btn-purple btn-sm" onClick={()=>setAddM(true)}>+ Novo Cliente</button>
      </div>
      <div className="content">
        {clients.length===0?<div className="empty" style={{marginTop:40}}><div className="empty-ico">👥</div><div className="empty-txt">Nenhum cliente ainda.<br/>Adicione o primeiro!</div></div>:(
          <div className="cli-layout">
            <div className="card">
              <div className="card-title">BASE DE CLIENTES</div>
              {clients.map(c=>(
                <div key={c.id} className={`cli-row ${sel?.id===c.id?"sel":""}`} onClick={()=>pick(c)}>
                  <div style={{flex:1}}>
                    <div className="cli-name">{c.nome}</div>
                    <div className="cli-meta">{c.compras||0}x compras · LTV {fmtS(c.ltv||0)}</div>
                    <div className="cli-ltv"><div className="cli-ltv-fill" style={{width:`${((c.ltv||0)/maxLTV)*100}%`}}/></div>
                  </div>
                  <div className="cli-total">{fmt(c.total||0)}</div>
                </div>
              ))}
            </div>
            {sel&&(
              <div className="card">
                <div className="card-title">FICHA — {sel.nome}</div>
                <div className="form-row"><div className="lbl">Nome/ID</div><input className="inp" value={nome} onChange={e=>setNome(e.target.value)}/></div>
                <div className="form-row" style={{marginTop:10}}><div className="lbl">Contato</div><input className="inp" value={contato} onChange={e=>setContato(e.target.value)}/></div>
                <div className="form-row" style={{marginTop:10}}><div className="lbl">Notas</div><textarea className="inp" style={{minHeight:68,resize:"none"}} value={notas} onChange={e=>setNotas(e.target.value)}/></div>
                <div style={{display:"flex",gap:8,marginTop:12}}>
                  <button className="btn btn-red btn-sm" onClick={async()=>{await sb.delete("clients",sel.id);await loadClients();setSel(null);}}>🗑</button>
                  <button className="btn btn-purple btn-sm" style={{flex:1}} onClick={async()=>{
                    await sb.update("clients",sel.id,{nome,contato,notas});
                    await loadClients();setSel(p=>({...p,nome,contato,notas}));
                  }}>✓ Salvar</button>
                </div>
                {cSales.length>0&&(
                  <div style={{marginTop:16}}>
                    <div className="card-title">HISTÓRICO</div>
                    {cSales.map(s=>(
                      <div key={s.id} className="hist-item">
                        <div><div className="hist-cnt">{s.conta}</div><div className="hist-dt">{s.data}</div></div>
                        <div className="hist-v">{fmt(s.valor)}</div>
                      </div>
                    ))}
                  </div>
                )}
              </div>
            )}
          </div>
        )}
      </div>
      {addM&&(
        <div className="overlay" onClick={()=>setAddM(false)}>
          <div className="modal" onClick={e=>e.stopPropagation()}>
            <div className="modal-title">NOVO <em>CLIENTE</em></div>
            <div className="form-row"><div className="lbl">Nome/ID</div><input className="inp" value={newN} onChange={e=>setNewN(e.target.value)}/></div>
            <div className="form-acts">
              <button className="btn btn-ghost" onClick={()=>setAddM(false)}>Cancelar</button>
              <button className="btn btn-purple" onClick={async()=>{
                if(!newN) return;
                await sb.insert("clients",{nome:newN,compras:0,total:0,contato:"",notas:"",ltv:0});
                await loadClients();setAddM(false);setNewN("");
              }}>✓ Adicionar</button>
            </div>
          </div>
        </div>
      )}
    </>
  );
}

/* ══════════ DK ADVISOR ═════════════════════════════════════════ */
function AdvisorPage({sales,accounts,clients,pipeline,profile}){
  const [msgs,setMsgs]=useState([{role:"assistant",content:`Olá, ${profile?.nome||""}! Sou o **DK Advisor**, IA exclusiva da **DK Academy by Raveli DK**.\n\nTenho acesso completo ao seu portfólio pessoal em tempo real. Pergunte qualquer coisa sobre suas vendas, contas, clientes ou estratégia.`}]);
  const [inp,setInp]=useState("");
  const [loading,setLoading]=useState(false);
  const bot=useRef(null);
  useEffect(()=>{bot.current?.scrollIntoView({behavior:"smooth"});},[msgs]);

  const ctx=`PORTFÓLIO DE ${profile?.nome||"usuário"}:
Faturamento: ${fmt(sales.reduce((a,s)=>a+(s.valor||0),0))}
Vendas: ${JSON.stringify(sales.slice(0,10))}
Contas disponíveis: ${JSON.stringify(accounts.filter(a=>a.status==="available").map(a=>({handle:a.handle,nicho:a.nicho,seguidores:a.seguidores,valor:a.valor,score:computeScore(a)})))}
Pipeline: ${JSON.stringify(pipeline.map(d=>({conta:d.conta,nicho:d.nicho,valor:d.valor,stage:d.stage,dias:d.dias})))}
Clientes: ${JSON.stringify(clients.map(c=>({nome:c.nome,compras:c.compras,total:c.total,ltv:c.ltv})))}`;

  const send=async(text)=>{
    const q=text||inp.trim();
    if(!q||loading) return;
    setInp("");
    const newMsgs=[...msgs,{role:"user",content:q}];
    setMsgs(newMsgs);setLoading(true);
    try{
      const res=await fetch("https://api.anthropic.com/v1/messages",{
        method:"POST",headers:{"Content-Type":"application/json"},
        body:JSON.stringify({
          model:"claude-sonnet-4-20250514",max_tokens:1200,
          system:`Você é o DK Advisor, IA exclusiva da DK Academy by Raveli DK. Use os dados reais do portfólio do usuário. Seja direto, use números reais, dê conselhos específicos e acionáveis. Use **negrito** para destacar. Máximo 4 parágrafos. Português brasileiro.\n\n${ctx}`,
          messages:newMsgs.map(m=>({role:m.role,content:m.content}))
        })
      });
      const data=await res.json();
      setMsgs(m=>[...m,{role:"assistant",content:data.content?.[0]?.text||"Erro."}]);
    }catch{setMsgs(m=>[...m,{role:"assistant",content:"Erro de conexão."}]);}
    setLoading(false);
  };

  const render=c=>c.split(/\*\*(.*?)\*\*/g).map((p,i)=>i%2===1?<strong key={i}>{p}</strong>:p);
  const SUGS=["Qual conta vender primeiro?","Como melhorar meu pipeline?","Previsão do próximo mês","Quais clientes reativar?","Análise do meu portfólio"];

  return(
    <>
      <div className="topbar">
        <div>
          <div className="pg-title">◈ DK <em>ADVISOR</em> — INTELIGÊNCIA IA</div>
          <div className="pg-sub">Portfólio de {profile?.nome||"usuário"} · claude-sonnet-4</div>
        </div>
        <div style={{display:"flex",alignItems:"center",gap:7,fontSize:11,fontFamily:"var(--mono)"}}>
          <div style={{width:7,height:7,borderRadius:"50%",background:"var(--up)",boxShadow:"0 0 8px var(--up)",animation:"blink 2s ease infinite"}}/>
          <span style={{color:"var(--up)"}}>online</span>
        </div>
      </div>
      <div className="chat-page">
        <div className="chat-hist">
          {msgs.map((m,i)=>(
            <div key={i} className={`msg ${m.role==="user"?"u":""}`}>
              <div className={`msg-av ${m.role==="user"?"u":"ai"}`}>{m.role==="user"?(profile?.nome?.slice(0,2)||"U").toUpperCase():"DK"}</div>
              <div className={`msg-bub ${m.role==="user"?"u":"ai"}`} style={{whiteSpace:"pre-wrap"}}>{render(m.content)}</div>
            </div>
          ))}
          {loading&&<div className="msg"><div className="msg-av ai">DK</div><div className="msg-bub ai"><div className="typing"><span/><span/><span/></div></div></div>}
          <div ref={bot}/>
        </div>
        <div className="sugs">{SUGS.map((s,i)=><span key={i} className="chip" onClick={()=>send(s)}>{s}</span>)}</div>
        <div className="chat-bar">
          <textarea className="chat-inp" rows={1} placeholder="Pergunte sobre seu portfólio…" value={inp} onChange={e=>setInp(e.target.value)}
            onKeyDown={e=>{if(e.key==="Enter"&&!e.shiftKey){e.preventDefault();send();}}}/>
          <button className="send-btn" onClick={()=>send()} disabled={loading||!inp.trim()}>➤</button>
        </div>
      </div>
    </>
  );
}

/* ══════════ SALE MODAL ═════════════════════════════════════════ */
function SaleModal({onClose,onSave,accounts}){
  const [form,setForm]=useState({conta:"",nicho:"",cliente:"",valor:""});
  const [saving,setSaving]=useState(false);
  const sf=(k,v)=>setForm(p=>({...p,[k]:v}));
  const avail=accounts.filter(a=>a.status==="available");
  return(
    <div className="overlay" onClick={onClose}>
      <div className="modal" onClick={e=>e.stopPropagation()}>
        <div className="modal-title">REGISTRAR <em>VENDA</em></div>
        <div className="form2">
          <div className="form-row"><div className="lbl">Conta</div>
            <select className="inp" value={form.conta} onChange={e=>{
              const a=avail.find(ac=>ac.handle===e.target.value);
              setForm(p=>({...p,conta:e.target.value,nicho:a?.nicho||p.nicho,valor:a?.valor||p.valor}));
            }}>
              <option value="">Selecionar...</option>
              {avail.map(a=><option key={a.id}>{a.handle}</option>)}
            </select>
          </div>
          <div className="form-row"><div className="lbl">Nicho</div>
            <select className="inp" value={form.nicho} onChange={e=>sf("nicho",e.target.value)}>
              <option value="">Selecionar...</option>{NICHOS.map(n=><option key={n}>{n}</option>)}
            </select>
          </div>
          <div className="form-row"><div className="lbl">Cliente</div><input className="inp" placeholder="Nome ou ID" value={form.cliente} onChange={e=>sf("cliente",e.target.value)}/></div>
          <div className="form-row"><div className="lbl">Valor (R$)</div><input className="inp" type="number" value={form.valor} onChange={e=>sf("valor",e.target.value)}/></div>
        </div>
        <div className="form-acts">
          <button className="btn btn-ghost" onClick={onClose}>Cancelar</button>
          <button className="btn btn-purple" disabled={saving} onClick={async()=>{
            if(!form.conta||!form.valor) return;
            setSaving(true);
            await onSave(form);
            setSaving(false);
          }}>✓ Confirmar Venda</button>
        </div>
      </div>
    </div>
  );
}

/* ══════════ ROOT APP ═══════════════════════════════════════════ */
export default function App(){
  const [configured,setConfigured]=useState(()=>{
    const url=localStorage.getItem("dk_sb_url");
    const key=localStorage.getItem("dk_sb_key");
    if(url&&key){sb.url=url;sb.key=key;return true;}
    return SUPABASE_URL!=="COLE_SUA_URL_AQUI";
  });
  const [authed,setAuthed]=useState(false);
  const [loading,setLoading]=useState(configured);
  const [page,setPage]=useState("dashboard");
  const [sales,setSales]=useState([]);
  const [accounts,setAccounts]=useState([]);
  const [clients,setClients]=useState([]);
  const [pipeline,setPipeline]=useState([]);
  const [profile,setProfile]=useState(null);
  const [saleM,setSaleM]=useState(false);

  const loadAll=useCallback(async()=>{
    const [s,a,c,p,pr]=await Promise.all([
      sb.select("sales"),
      sb.select("accounts"),
      sb.select("clients"),
      sb.select("pipeline_deals"),
      sb.getProfile(),
    ]);
    if(Array.isArray(s)) setSales(s);
    if(Array.isArray(a)) setAccounts(a);
    if(Array.isArray(c)) setClients(c);
    if(Array.isArray(p)) setPipeline(p);
    if(pr) setProfile(pr);
  },[]);

  // Verificar sessão salva
  useEffect(()=>{
    if(!configured){setLoading(false);return;}
    const token=localStorage.getItem("dk_token");
    const uid=localStorage.getItem("dk_uid");
    if(token&&uid){
      sb.token=token;sb.userId=uid;
      loadAll().then(()=>{setAuthed(true);setLoading(false);})
        .catch(()=>{setLoading(false);});
    } else {setLoading(false);}
  },[configured]);

  // Watch auth state via polling
  useEffect(()=>{
    if(!configured) return;
    const check=setInterval(()=>{
      if(sb.token&&!authed){
        localStorage.setItem("dk_token",sb.token);
        localStorage.setItem("dk_uid",sb.userId||"");
        loadAll().then(()=>setAuthed(true));
      }
    },500);
    return()=>clearInterval(check);
  },[authed,configured]);

  const handleSale=async(f)=>{
    if(!f.conta||!f.valor) return;
    await sb.insert("sales",{data:todayStr(),conta:f.conta,nicho:f.nicho,cliente:f.cliente,valor:Number(f.valor),status:"paid"});
    const acct=accounts.find(a=>a.handle===f.conta);
    if(acct) await sb.update("accounts",acct.id,{status:"sold"});
    const cli=clients.find(c=>c.nome===f.cliente);
    if(cli){
      await sb.update("clients",cli.id,{compras:(cli.compras||0)+1,total:(cli.total||0)+Number(f.valor)});
    }
    await loadAll();setSaleM(false);
  };

  const handleGoalUpdate=async(val)=>{
    if(!val) return;
    await sb.updateProfile({meta_mensal:val});
    setProfile(p=>({...p,meta_mensal:val}));
  };

  const logout=()=>{
    sb.signOut();localStorage.removeItem("dk_token");localStorage.removeItem("dk_uid");
    setAuthed(false);setSales([]);setAccounts([]);setClients([]);setPipeline([]);setProfile(null);
  };

  const totalRev=sales.reduce((a,s)=>a+(s.valor||0),0);
  const NAV=[
    {id:"dashboard",ico:"◈",lbl:"Dashboard",badge:0},
    {id:"pipeline", ico:"⬡",lbl:"Pipeline CRM",badge:pipeline.length,bc:"purple"},
    {id:"contas",   ico:"◉",lbl:"Portfólio",badge:accounts.filter(a=>a.status==="available").length,bc:"green"},
    {id:"clientes", ico:"◆",lbl:"Clientes",badge:clients.length,bc:"purple"},
    {id:"advisor",  ico:"✦",lbl:"DK Advisor IA",badge:0},
  ];

  if(!configured) return(
    <>
      <Styles/>
      <SetupScreen onSave={(url,key)=>{
        sb.url=url;sb.key=key;
        localStorage.setItem("dk_sb_url",url);
        localStorage.setItem("dk_sb_key",key);
        setConfigured(true);setLoading(false);
      }}/>
    </>
  );

  if(loading) return(
    <>
      <Styles/>
      <div className="loading-screen">
        <div className="spinner"/>
        <div style={{fontFamily:"var(--head)",fontSize:12,color:"var(--t3)",letterSpacing:2}}>DK ACADEMY</div>
      </div>
    </>
  );

  if(!authed) return <><Styles/><AuthScreen/></>;

  return(
    <>
      <Styles/>
      <div className="app">
        <aside className="sb">
          <div className="sb-brand">
            <div className="brand-row">
              <div className="brand-icon">DK</div>
              <div><div className="brand-name">DK ACADEMY</div><div className="brand-by">by Raveli DK</div></div>
            </div>
            <div className="brand-live">
              <div className="live-dot"/><div className="live-txt">ao vivo</div>
              <div className="live-val">{fmtS(totalRev)}</div>
            </div>
          </div>
          <nav className="sb-nav">
            <div className="sb-sec">Principal</div>
            {NAV.map(item=>(
              <div key={item.id} className={`sb-item ${page===item.id?"on":""}`} onClick={()=>setPage(item.id)}>
                <span className="sb-ico">{item.ico}</span>{item.lbl}
                {item.badge>0&&<span className={`sb-badge ${item.bc||"purple"}`}>{item.badge}</span>}
              </div>
            ))}
            <div className="sb-sec" style={{marginTop:8}}>Conta</div>
            <div className={`sb-item ${page==="config"?"on":""}`} onClick={()=>setPage("config")}>
              <span className="sb-ico">⚙</span>Configurações
            </div>
            <div style={{flex:1}}/>
            <div className="sb-ai" onClick={()=>setPage("advisor")}>
              <div className="sb-ai-title">◈ DK ADVISOR IA</div>
              <div className="sb-ai-sub">IA com acesso ao seu portfólio em tempo real</div>
            </div>
          </nav>
          <div className="sb-foot">
            <div className="sb-avatar">{(profile?.nome||"U").slice(0,2).toUpperCase()}</div>
            <div><div className="sb-uname">{profile?.nome||"Usuário"}</div><div className="sb-uplan">DK ACADEMY · PRO</div></div>
            <button className="sb-logout" onClick={logout} title="Sair">⏻</button>
          </div>
        </aside>

        <main className="main">
          {page==="dashboard"&&<Dashboard sales={sales} accounts={accounts} pipeline={pipeline} profile={profile} onNewSale={()=>setSaleM(true)} onGoalUpdate={handleGoalUpdate}/>}
          {page==="pipeline" &&<PipelinePage pipeline={pipeline} loadPipe={async()=>{const p=await sb.select("pipeline_deals");if(Array.isArray(p))setPipeline(p);}}/>}
          {page==="contas"   &&<ContasPage accounts={accounts} loadAccounts={async()=>{const a=await sb.select("accounts");if(Array.isArray(a))setAccounts(a);}}/>}
          {page==="clientes" &&<ClientesPage clients={clients} sales={sales} loadClients={async()=>{const c=await sb.select("clients");if(Array.isArray(c))setClients(c);}}/>}
          {page==="advisor"  &&<AdvisorPage sales={sales} accounts={accounts} clients={clients} pipeline={pipeline} profile={profile}/>}
          {page==="config"   &&(
            <>
              <div className="topbar"><div className="pg-title">⚙ <em>CONFIGURAÇÕES</em></div></div>
              <div className="content">
                <div className="card" style={{maxWidth:460}}>
                  <div className="card-title">MEU PERFIL</div>
                  <div className="form-row"><div className="lbl">Nome</div>
                    <input className="inp" defaultValue={profile?.nome||""} id="cfg-nome"/>
                  </div>
                  <div className="form-row" style={{marginTop:10}}><div className="lbl">Empresa</div>
                    <input className="inp" defaultValue={profile?.empresa||"DK Academy"} id="cfg-emp"/>
                  </div>
                  <div style={{marginTop:14,display:"flex",gap:8}}>
                    <button className="btn btn-purple" onClick={async()=>{
                      const n=document.getElementById("cfg-nome")?.value;
                      const e=document.getElementById("cfg-emp")?.value;
                      await sb.updateProfile({nome:n,empresa:e});
                      setProfile(p=>({...p,nome:n,empresa:e}));
                    }}>✓ Salvar Perfil</button>
                    <button className="btn btn-red" onClick={logout}>Sair da Conta</button>
                  </div>
                </div>
                <div className="card" style={{maxWidth:460}}>
                  <div className="card-title">SOBRE O PLANO</div>
                  <div style={{fontSize:13,color:"var(--t2)",lineHeight:1.6}}>
                    <div style={{color:"var(--dk-xl)",fontWeight:700,marginBottom:8}}>DK Academy — Plano PRO</div>
                    Todos os seus dados são armazenados com segurança no Supabase. Cada usuário tem acesso exclusivo às suas próprias informações com isolamento total (Row Level Security).
                  </div>
                </div>
              </div>
            </>
          )}
        </main>
      </div>
      {saleM&&<SaleModal onClose={()=>setSaleM(false)} onSave={handleSale} accounts={accounts}/>}
    </>
  );
}
