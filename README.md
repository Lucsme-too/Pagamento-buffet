import { useState, useEffect, useCallback } from "react";

// ─── Mock Data ────────────────────────────────────────────────────────────────
const INITIAL_CLIENTS = [
  {
    id: "c1",
    name: "Ana Souza",
    email: "ana@email.com",
    password: "ana123",
    event: "Casamento",
    eventDate: "2026-06-15",
    total: 12000,
    payments: [
      { id: "p1", date: "2026-01-10", amount: 3000, status: "pago", method: "PIX", note: "Entrada" },
      { id: "p2", date: "2026-03-10", amount: 3000, status: "pago", method: "Transferência", note: "2ª parcela" },
      { id: "p3", date: "2026-05-10", amount: 3000, status: "pendente", method: "", note: "3ª parcela" },
      { id: "p4", date: "2026-06-10", amount: 3000, status: "pendente", method: "", note: "Final" },
    ],
  },
  {
    id: "c2",
    name: "Carlos Lima",
    email: "carlos@email.com",
    password: "carlos123",
    event: "Aniversário 15 anos",
    eventDate: "2026-07-20",
    total: 8500,
    payments: [
      { id: "p5", date: "2026-02-01", amount: 2500, status: "pago", method: "Cartão", note: "Sinal" },
      { id: "p6", date: "2026-04-01", amount: 3000, status: "pendente", method: "", note: "2ª parcela" },
      { id: "p7", date: "2026-07-01", amount: 3000, status: "pendente", method: "", note: "Saldo final" },
    ],
  },
  {
    id: "c3",
    name: "Beatriz Mendes",
    email: "beatriz@email.com",
    password: "bea123",
    event: "Formatura",
    eventDate: "2026-12-05",
    total: 6000,
    payments: [
      { id: "p8", date: "2026-03-15", amount: 2000, status: "pago", method: "PIX", note: "Entrada" },
      { id: "p9", date: "2026-07-15", amount: 2000, status: "pendente", method: "", note: "2ª parcela" },
      { id: "p10", date: "2026-11-15", amount: 2000, status: "pendente", method: "", note: "Final" },
    ],
  },
];

const SUPPLIER = { email: "buffet@admin.com", password: "admin2026" };

// ─── Helpers ──────────────────────────────────────────────────────────────────
const fmt = (v) => v.toLocaleString("pt-BR", { style: "currency", currency: "BRL" });
const uid = () => Math.random().toString(36).slice(2, 9);

// ─── Styles ───────────────────────────────────────────────────────────────────
const css = `
  @import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,300;0,400;0,600;1,300;1,400&family=DM+Mono:wght@300;400;500&display=swap');

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --ink:    #0d0d0d;
    --paper:  #f7f4ef;
    --cream:  #ede9e0;
    --gold:   #b8922a;
    --gold2:  #d4a93a;
    --rust:   #c0392b;
    --sage:   #4a7c59;
    --muted:  #7a7060;
    --border: rgba(184,146,42,0.25);
  }

  html, body, #root { height: 100%; font-family: 'DM Mono', monospace; background: var(--paper); color: var(--ink); }

  /* ── scrollbar ── */
  ::-webkit-scrollbar { width: 4px; }
  ::-webkit-scrollbar-track { background: var(--cream); }
  ::-webkit-scrollbar-thumb { background: var(--gold); border-radius: 2px; }

  /* ── login ── */
  .login-wrap {
    min-height: 100vh;
    display: grid;
    grid-template-columns: 1fr 1fr;
    background: var(--ink);
  }
  @media (max-width: 768px) { .login-wrap { grid-template-columns: 1fr; } }

  .login-art {
    background: linear-gradient(135deg, #1a150a 0%, #2e2210 50%, #1a150a 100%);
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    padding: 3rem;
    position: relative; overflow: hidden;
  }
  .login-art::before {
    content: '';
    position: absolute; inset: 0;
    background: repeating-linear-gradient(0deg, transparent, transparent 39px, rgba(184,146,42,0.06) 40px),
                repeating-linear-gradient(90deg, transparent, transparent 39px, rgba(184,146,42,0.06) 40px);
  }
  .login-art-title {
    font-family: 'Cormorant Garamond', serif;
    font-size: clamp(2.5rem, 5vw, 4rem);
    color: var(--gold2);
    line-height: 1.1;
    text-align: center;
    position: relative; z-index: 1;
  }
  .login-art-sub {
    font-size: 0.7rem;
    color: rgba(212,169,58,0.5);
    letter-spacing: 0.25em;
    text-transform: uppercase;
    margin-top: 0.75rem;
    position: relative; z-index: 1;
  }
  .login-art-ornament {
    width: 60px; height: 1px;
    background: var(--gold);
    margin: 1.5rem auto;
    position: relative; z-index: 1;
  }

  .login-form-side {
    display: flex; align-items: center; justify-content: center;
    padding: 3rem 2rem;
    background: #111;
  }
  .login-card {
    width: 100%; max-width: 380px;
  }
  .login-tabs {
    display: flex; gap: 0;
    border: 1px solid var(--border);
    border-radius: 2px;
    overflow: hidden;
    margin-bottom: 2rem;
  }
  .login-tab {
    flex: 1; padding: 0.75rem;
    background: transparent;
    border: none; cursor: pointer;
    font-family: 'DM Mono', monospace;
    font-size: 0.7rem;
    letter-spacing: 0.15em;
    text-transform: uppercase;
    color: var(--muted);
    transition: all 0.2s;
  }
  .login-tab.active { background: var(--gold); color: var(--ink); font-weight: 500; }
  .login-tab:not(.active):hover { background: rgba(184,146,42,0.1); color: var(--gold); }

  .field { margin-bottom: 1.25rem; }
  .field label {
    display: block;
    font-size: 0.65rem;
    letter-spacing: 0.2em;
    text-transform: uppercase;
    color: rgba(184,146,42,0.7);
    margin-bottom: 0.4rem;
  }
  .field input, .field select, .field textarea {
    width: 100%;
    background: rgba(255,255,255,0.04);
    border: 1px solid rgba(184,146,42,0.2);
    border-radius: 2px;
    padding: 0.75rem 1rem;
    color: #e8e0d0;
    font-family: 'DM Mono', monospace;
    font-size: 0.85rem;
    transition: border-color 0.2s;
    outline: none;
  }
  .field input:focus, .field select:focus, .field textarea:focus {
    border-color: var(--gold);
    background: rgba(184,146,42,0.05);
  }
  .field input::placeholder { color: rgba(184,146,42,0.3); }

  .btn {
    padding: 0.75rem 1.5rem;
    border: none; border-radius: 2px;
    cursor: pointer;
    font-family: 'DM Mono', monospace;
    font-size: 0.75rem;
    letter-spacing: 0.15em;
    text-transform: uppercase;
    transition: all 0.2s;
  }
  .btn-gold {
    background: var(--gold);
    color: var(--ink);
    width: 100%;
  }
  .btn-gold:hover { background: var(--gold2); transform: translateY(-1px); }
  .btn-outline {
    background: transparent;
    border: 1px solid var(--border);
    color: var(--gold);
  }
  .btn-outline:hover { border-color: var(--gold); background: rgba(184,146,42,0.07); }
  .btn-ghost {
    background: transparent; border: none;
    color: var(--muted); font-size: 0.7rem;
    padding: 0.4rem 0.75rem;
    cursor: pointer;
    font-family: 'DM Mono', monospace;
  }
  .btn-ghost:hover { color: var(--gold); }
  .btn-danger {
    background: transparent; border: 1px solid rgba(192,57,43,0.3);
    color: var(--rust); font-size: 0.7rem;
    padding: 0.4rem 0.75rem;
  }
  .btn-danger:hover { background: rgba(192,57,43,0.1); }
  .btn-sm { padding: 0.4rem 0.9rem; font-size: 0.65rem; }
  .btn-success { background: var(--sage); color: #fff; }
  .btn-success:hover { filter: brightness(1.15); }

  .err-msg {
    color: #e74c3c; font-size: 0.7rem;
    margin-top: 0.75rem; text-align: center;
    letter-spacing: 0.05em;
  }

  /* ── app shell ── */
  .app { min-height: 100vh; display: flex; flex-direction: column; }

  .topbar {
    background: var(--ink);
    border-bottom: 1px solid var(--border);
    padding: 0 2rem;
    height: 56px;
    display: flex; align-items: center; justify-content: space-between;
    position: sticky; top: 0; z-index: 100;
  }
  .topbar-brand {
    font-family: 'Cormorant Garamond', serif;
    font-size: 1.3rem;
    color: var(--gold);
    letter-spacing: 0.05em;
  }
  .topbar-right { display: flex; align-items: center; gap: 1rem; }
  .topbar-user { font-size: 0.7rem; color: rgba(184,146,42,0.6); letter-spacing: 0.1em; }
  .badge-role {
    font-size: 0.6rem; letter-spacing: 0.15em; text-transform: uppercase;
    padding: 0.2rem 0.6rem; border-radius: 99px;
    background: rgba(184,146,42,0.15); color: var(--gold);
    border: 1px solid var(--border);
  }

  .main { flex: 1; padding: 2.5rem 2rem; max-width: 1100px; margin: 0 auto; width: 100%; }

  /* ── cards ── */
  .card {
    background: #fff;
    border: 1px solid #e8e2d8;
    border-radius: 4px;
    padding: 1.75rem;
    box-shadow: 0 2px 12px rgba(0,0,0,0.05);
  }
  .card + .card { margin-top: 1.5rem; }

  .section-title {
    font-family: 'Cormorant Garamond', serif;
    font-size: 1.6rem;
    font-weight: 400;
    color: var(--ink);
    margin-bottom: 0.25rem;
  }
  .section-sub {
    font-size: 0.7rem;
    color: var(--muted);
    letter-spacing: 0.1em;
    text-transform: uppercase;
    margin-bottom: 1.75rem;
  }

  /* ── stats row ── */
  .stats-row { display: grid; grid-template-columns: repeat(auto-fit, minmax(160px, 1fr)); gap: 1rem; margin-bottom: 1.5rem; }
  .stat-box {
    background: var(--cream);
    border: 1px solid #ddd6c8;
    border-radius: 3px;
    padding: 1.25rem 1rem;
  }
  .stat-box .stat-val {
    font-family: 'Cormorant Garamond', serif;
    font-size: 1.5rem;
    color: var(--ink);
    line-height: 1;
  }
  .stat-box .stat-lbl {
    font-size: 0.6rem;
    text-transform: uppercase;
    letter-spacing: 0.2em;
    color: var(--muted);
    margin-top: 0.35rem;
  }
  .stat-box.gold { border-color: rgba(184,146,42,0.4); background: rgba(184,146,42,0.06); }
  .stat-box.gold .stat-val { color: var(--gold); }
  .stat-box.green { border-color: rgba(74,124,89,0.3); background: rgba(74,124,89,0.06); }
  .stat-box.green .stat-val { color: var(--sage); }
  .stat-box.red { border-color: rgba(192,57,43,0.3); background: rgba(192,57,43,0.06); }
  .stat-box.red .stat-val { color: var(--rust); }

  /* ── progress bar ── */
  .progress-wrap { margin: 0.5rem 0; }
  .progress-bar-bg {
    background: #e8e2d8; border-radius: 99px; height: 6px; overflow: hidden;
  }
  .progress-bar-fill {
    height: 100%; border-radius: 99px;
    background: linear-gradient(90deg, var(--gold), var(--gold2));
    transition: width 0.6s ease;
  }
  .progress-label { font-size: 0.65rem; color: var(--muted); margin-top: 0.3rem; }

  /* ── table ── */
  .tbl-wrap { overflow-x: auto; margin-top: 1rem; }
  table { width: 100%; border-collapse: collapse; }
  th {
    background: var(--cream);
    font-size: 0.62rem;
    text-transform: uppercase;
    letter-spacing: 0.18em;
    color: var(--muted);
    padding: 0.6rem 1rem;
    text-align: left;
    border-bottom: 1px solid #ddd6c8;
    white-space: nowrap;
  }
  td {
    padding: 0.8rem 1rem;
    font-size: 0.8rem;
    border-bottom: 1px solid #f0ebe3;
    vertical-align: middle;
  }
  tr:last-child td { border-bottom: none; }
  tr:hover td { background: rgba(184,146,42,0.03); }

  /* ── status pill ── */
  .pill {
    display: inline-block;
    font-size: 0.6rem;
    letter-spacing: 0.12em;
    text-transform: uppercase;
    padding: 0.25rem 0.7rem;
    border-radius: 99px;
    font-weight: 500;
  }
  .pill-pago { background: rgba(74,124,89,0.12); color: var(--sage); border: 1px solid rgba(74,124,89,0.25); }
  .pill-pendente { background: rgba(184,146,42,0.12); color: var(--gold); border: 1px solid rgba(184,146,42,0.25); }
  .pill-atrasado { background: rgba(192,57,43,0.12); color: var(--rust); border: 1px solid rgba(192,57,43,0.25); }

  /* ── modal ── */
  .modal-overlay {
    position: fixed; inset: 0;
    background: rgba(13,13,13,0.7);
    backdrop-filter: blur(4px);
    z-index: 200;
    display: flex; align-items: center; justify-content: center;
    padding: 1rem;
    animation: fadeIn 0.2s;
  }
  @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
  .modal {
    background: #fff;
    border-radius: 4px;
    padding: 2rem;
    width: 100%; max-width: 480px;
    box-shadow: 0 20px 60px rgba(0,0,0,0.3);
    animation: slideUp 0.25s ease;
  }
  @keyframes slideUp { from { transform: translateY(20px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
  .modal-title {
    font-family: 'Cormorant Garamond', serif;
    font-size: 1.4rem;
    margin-bottom: 1.5rem;
    color: var(--ink);
  }
  .modal-footer { display: flex; gap: 0.75rem; justify-content: flex-end; margin-top: 1.5rem; }

  /* ── admin client list ── */
  .client-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); gap: 1rem; }
  .client-card {
    background: #fff;
    border: 1px solid #e8e2d8;
    border-radius: 4px;
    padding: 1.25rem;
    cursor: pointer;
    transition: border-color 0.2s, box-shadow 0.2s;
  }
  .client-card:hover { border-color: var(--gold); box-shadow: 0 4px 20px rgba(184,146,42,0.12); }
  .client-card.active { border-color: var(--gold); box-shadow: 0 0 0 2px rgba(184,146,42,0.15); }
  .client-name { font-family: 'Cormorant Garamond', serif; font-size: 1.2rem; margin-bottom: 0.2rem; }
  .client-meta { font-size: 0.7rem; color: var(--muted); letter-spacing: 0.05em; }

  /* ── nav tabs ── */
  .nav-tabs { display: flex; gap: 0; border-bottom: 1px solid #e8e2d8; margin-bottom: 2rem; }
  .nav-tab {
    padding: 0.7rem 1.25rem;
    font-size: 0.7rem;
    letter-spacing: 0.12em;
    text-transform: uppercase;
    background: none; border: none;
    cursor: pointer;
    color: var(--muted);
    border-bottom: 2px solid transparent;
    transition: all 0.2s;
    font-family: 'DM Mono', monospace;
    margin-bottom: -1px;
  }
  .nav-tab.active { color: var(--gold); border-bottom-color: var(--gold); }
  .nav-tab:hover:not(.active) { color: var(--ink); }

  /* ── search ── */
  .search-row { display: flex; gap: 0.75rem; margin-bottom: 1.5rem; align-items: center; flex-wrap: wrap; }
  .search-input {
    background: #fff;
    border: 1px solid #ddd6c8;
    border-radius: 2px;
    padding: 0.6rem 1rem;
    font-family: 'DM Mono', monospace;
    font-size: 0.8rem;
    color: var(--ink);
    outline: none;
    flex: 1; min-width: 200px;
    transition: border-color 0.2s;
  }
  .search-input:focus { border-color: var(--gold); }
  .search-input::placeholder { color: #bbb; }

  .inline-form { display: flex; gap: 0.75rem; flex-wrap: wrap; align-items: flex-end; }
  .inline-form .field { margin-bottom: 0; flex: 1; min-width: 130px; }
  .inline-form .field label { color: var(--muted); }
  .inline-form .field input, .inline-form .field select {
    background: var(--cream); border-color: #ddd6c8; color: var(--ink;
  }

  .divider { height: 1px; background: #e8e2d8; margin: 1.5rem 0; }

  .empty-state {
    text-align: center; padding: 3rem;
    color: var(--muted); font-size: 0.8rem;
    letter-spacing: 0.1em;
  }

  .highlight-row td { background: rgba(184,146,42,0.05) !important; }
  
  .actions-cell { display: flex; gap: 0.4rem; flex-wrap: wrap; }
`;

// ─── App ──────────────────────────────────────────────────────────────────────
export default function App() {
  const [clients, setClients] = useState(null);
  const [session, setSession] = useState(null); // { role: 'client'|'supplier', id?, name? }
  const [loading, setLoading] = useState(true);

  // Load persisted data
  useEffect(() => {
    (async () => {
      try {
        const res = await window.storage.get("buffet-clients");
        setClients(res ? JSON.parse(res.value) : INITIAL_CLIENTS);
      } catch {
        setClients(INITIAL_CLIENTS);
      }
      setLoading(false);
    })();
  }, []);

  const persist = useCallback(async (updated) => {
    setClients(updated);
    try { await window.storage.set("buffet-clients", JSON.stringify(updated)); } catch {}
  }, []);

  if (loading) return (
    <div style={{ minHeight: "100vh", display: "flex", alignItems: "center", justifyContent: "center", fontFamily: "DM Mono, monospace", fontSize: "0.8rem", color: "#b8922a", background: "#0d0d0d" }}>
      carregando…
    </div>
  );

  return (
    <>
      <style>{css}</style>
      {!session
        ? <LoginPage clients={clients} onLogin={setSession} />
        : session.role === "supplier"
          ? <SupplierDashboard clients={clients} persist={persist} onLogout={() => setSession(null)} />
          : <ClientDashboard
              client={clients.find(c => c.id === session.id)}
              allClients={clients}
              persist={persist}
              session={session}
              onLogout={() => setSession(null)}
            />
      }
    </>
  );
}

// ─── Login ────────────────────────────────────────────────────────────────────
function LoginPage({ clients, onLogin }) {
  const [tab, setTab] = useState("client");
  const [email, setEmail] = useState("");
  const [pw, setPw] = useState("");
  const [err, setErr] = useState("");

  const handle = () => {
    setErr("");
    if (tab === "supplier") {
      if (email === SUPPLIER.email && pw === SUPPLIER.password) {
        onLogin({ role: "supplier", name: "Administrador" });
      } else { setErr("Credenciais inválidas."); }
    } else {
      const c = clients.find(x => x.email === email && x.password === pw);
      if (c) { onLogin({ role: "client", id: c.id, name: c.name }); }
      else { setErr("E-mail ou senha incorretos."); }
    }
  };

  return (
    <div className="login-wrap">
      <div className="login-art">
        <div className="login-art-sub">Sistema de Gestão</div>
        <div className="login-art-ornament" />
        <div className="login-art-title">Buffet<br /><em>Pagamentos</em></div>
        <div className="login-art-ornament" />
        <div className="login-art-sub" style={{ opacity: 0.4 }}>controle financeiro de eventos</div>
      </div>
      <div className="login-form-side">
        <div className="login-card">
          <div className="login-tabs">
            <button className={`login-tab${tab === "client" ? " active" : ""}`} onClick={() => { setTab("client"); setErr(""); }}>Cliente</button>
            <button className={`login-tab${tab === "supplier" ? " active" : ""}`} onClick={() => { setTab("supplier"); setErr(""); }}>Fornecedor</button>
          </div>
          <div className="field">
            <label>E-mail</label>
            <input type="email" placeholder={tab === "supplier" ? "buffet@admin.com" : "seu@email.com"} value={email} onChange={e => setEmail(e.target.value)} onKeyDown={e => e.key === "Enter" && handle()} />
          </div>
          <div className="field">
            <label>Senha</label>
            <input type="password" placeholder="••••••••" value={pw} onChange={e => setPw(e.target.value)} onKeyDown={e => e.key === "Enter" && handle()} />
          </div>
          {err && <div className="err-msg">{err}</div>}
          <button className="btn btn-gold" style={{ marginTop: "1.25rem" }} onClick={handle}>Entrar</button>
          {tab === "client" && (
            <div style={{ marginTop: "1rem", fontSize: "0.65rem", color: "rgba(184,146,42,0.4)", textAlign: "center", lineHeight: 1.8 }}>
              Teste: ana@email.com / ana123<br />carlos@email.com / carlos123
            </div>
          )}
        </div>
      </div>
    </div>
  );
}

// ─── Client Dashboard ─────────────────────────────────────────────────────────
function ClientDashboard({ client, allClients, persist, session, onLogout }) {
  const [showPayModal, setShowPayModal] = useState(null); // payment object
  const [payData, setPayData] = useState({ method: "", note: "" });

  if (!client) return <div className="empty-state">Cliente não encontrado.</div>;

  const paid = client.payments.filter(p => p.status === "pago").reduce((s, p) => s + p.amount, 0);
  const pending = client.total - paid;
  const pct = Math.round((paid / client.total) * 100);

  const openPay = (p) => { setPayData({ method: "", note: p.note }); setShowPayModal(p); };

  const confirmPay = () => {
    if (!payData.method) return;
    const updated = allClients.map(c =>
      c.id !== client.id ? c : {
        ...c,
        payments: c.payments.map(p =>
          p.id !== showPayModal.id ? p : { ...p, status: "pago", method: payData.method, note: payData.note, datePaid: new Date().toISOString().slice(0, 10) }
        )
      }
    );
    persist(updated);
    setShowPayModal(null);
  };

  return (
    <div className="app">
      <Topbar name={session.name} role="Cliente" onLogout={onLogout} />
      <div className="main">
        <div className="section-title">{client.name}</div>
        <div className="section-sub">{client.event} · {fmtDate(client.eventDate)}</div>

        <div className="stats-row">
          <div className="stat-box gold">
            <div className="stat-val">{fmt(client.total)}</div>
            <div className="stat-lbl">Total do contrato</div>
          </div>
          <div className="stat-box green">
            <div className="stat-val">{fmt(paid)}</div>
            <div className="stat-lbl">Total pago</div>
          </div>
          <div className="stat-box red">
            <div className="stat-val">{fmt(pending)}</div>
            <div className="stat-lbl">Saldo restante</div>
          </div>
        </div>

        <div className="card">
          <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: "0.5rem" }}>
            <span style={{ fontSize: "0.7rem", color: "var(--muted)", textTransform: "uppercase", letterSpacing: "0.12em" }}>Progresso do pagamento</span>
            <span style={{ fontSize: "0.8rem", color: "var(--gold)", fontWeight: 500 }}>{pct}%</span>
          </div>
          <div className="progress-bar-bg"><div className="progress-bar-fill" style={{ width: pct + "%" }} /></div>
          <div className="progress-label">{client.payments.filter(p => p.status === "pago").length} de {client.payments.length} parcelas quitadas</div>
        </div>

        <div className="card" style={{ marginTop: "1.5rem" }}>
          <div className="section-title" style={{ fontSize: "1.2rem", marginBottom: "1.5rem" }}>Minhas Parcelas</div>
          <div className="tbl-wrap">
            <table>
              <thead>
                <tr>
                  <th>Vencimento</th>
                  <th>Descrição</th>
                  <th>Valor</th>
                  <th>Método</th>
                  <th>Status</th>
                  <th>Ação</th>
                </tr>
              </thead>
              <tbody>
                {client.payments.map(p => (
                  <tr key={p.id} className={p.status === "pago" ? "highlight-row" : ""}>
                    <td>{fmtDate(p.date)}</td>
                    <td>{p.note}</td>
                    <td style={{ fontWeight: 500 }}>{fmt(p.amount)}</td>
                    <td style={{ color: "var(--muted)" }}>{p.method || "—"}</td>
                    <td><span className={`pill pill-${p.status}`}>{p.status}</span></td>
                    <td>
                      {p.status === "pendente"
                        ? <button className="btn btn-sm btn-success" onClick={() => openPay(p)}>Registrar pagamento</button>
                        : <span style={{ fontSize: "0.7rem", color: "var(--sage)" }}>✓ Pago em {p.datePaid ? fmtDate(p.datePaid) : "—"}</span>
                      }
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      </div>

      {showPayModal && (
        <div className="modal-overlay" onClick={e => e.target === e.currentTarget && setShowPayModal(null)}>
          <div className="modal">
            <div className="modal-title">Registrar Pagamento</div>
            <div style={{ fontSize: "0.8rem", color: "var(--muted)", marginBottom: "1.5rem" }}>
              {showPayModal.note} · <strong style={{ color: "var(--ink)" }}>{fmt(showPayModal.amount)}</strong>
            </div>
            <div className="field">
              <label>Forma de pagamento *</label>
              <select value={payData.method} onChange={e => setPayData(d => ({ ...d, method: e.target.value }))} style={{ background: "var(--cream)", borderColor: "#ddd6c8", color: "var(--ink)" }}>
                <option value="">Selecione…</option>
                {["PIX", "Transferência", "Cartão de Crédito", "Cartão de Débito", "Dinheiro", "Boleto"].map(m => <option key={m}>{m}</option>)}
              </select>
            </div>
            <div className="field">
              <label>Observação</label>
              <input value={payData.note} onChange={e => setPayData(d => ({ ...d, note: e.target.value }))} style={{ background: "var(--cream)", borderColor: "#ddd6c8", color: "var(--ink)" }} />
            </div>
            {!payData.method && <div style={{ fontSize: "0.7rem", color: "var(--rust)", marginBottom: "0.5rem" }}>Selecione a forma de pagamento</div>}
            <div className="modal-footer">
              <button className="btn btn-ghost" onClick={() => setShowPayModal(null)}>Cancelar</button>
              <button className="btn btn-gold btn-sm" style={{ width: "auto" }} onClick={confirmPay}>Confirmar</button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}

// ─── Supplier Dashboard ───────────────────────────────────────────────────────
function SupplierDashboard({ clients, persist, onLogout }) {
  const [tab, setTab] = useState("overview");
  const [selected, setSelected] = useState(null);
  const [search, setSearch] = useState("");
  const [modal, setModal] = useState(null); // 'addClient' | 'editClient' | 'addPayment' | 'editPayment' | 'deleteClient' | 'deletePayment'
  const [form, setForm] = useState({});

  const totalRevenue = clients.reduce((s, c) => s + c.total, 0);
  const totalPaid = clients.reduce((s, c) => s + c.payments.filter(p => p.status === "pago").reduce((x, p) => x + p.amount, 0), 0);
  const totalPending = totalRevenue - totalPaid;
  const allPayments = clients.flatMap(c => c.payments.map(p => ({ ...p, clientName: c.name, clientId: c.id })));

  const filtered = clients.filter(c =>
    c.name.toLowerCase().includes(search.toLowerCase()) ||
    c.event.toLowerCase().includes(search.toLowerCase()) ||
    c.email.toLowerCase().includes(search.toLowerCase())
  );

  // ── helpers ──
  const openModal = (type, data = {}) => { setForm(data); setModal(type); };

  const saveClient = () => {
    if (!form.name || !form.email || !form.event) return;
    if (modal === "addClient") {
      persist([...clients, { id: "c" + uid(), name: form.name, email: form.email, password: form.password || "senha123", event: form.event, eventDate: form.eventDate || "", total: Number(form.total) || 0, payments: [] }]);
    } else {
      persist(clients.map(c => c.id !== form.id ? c : { ...c, name: form.name, email: form.email, event: form.event, eventDate: form.eventDate, total: Number(form.total) }));
      if (selected?.id === form.id) setSelected(clients.find(c => c.id === form.id));
    }
    setModal(null);
  };

  const deleteClient = () => {
    persist(clients.filter(c => c.id !== form.id));
    if (selected?.id === form.id) setSelected(null);
    setModal(null);
  };

  const savePayment = () => {
    if (!form.amount || !form.date) return;
    if (modal === "addPayment") {
      const newP = { id: "p" + uid(), date: form.date, amount: Number(form.amount), status: form.status || "pendente", method: form.method || "", note: form.note || "" };
      persist(clients.map(c => c.id !== form.clientId ? c : { ...c, payments: [...c.payments, newP] }));
    } else {
      persist(clients.map(c => c.id !== form.clientId ? c : {
        ...c, payments: c.payments.map(p => p.id !== form.id ? p : { ...p, date: form.date, amount: Number(form.amount), status: form.status, method: form.method, note: form.note })
      }));
    }
    setModal(null);
  };

  const deletePayment = () => {
    persist(clients.map(c => c.id !== form.clientId ? c : { ...c, payments: c.payments.filter(p => p.id !== form.id) }));
    setModal(null);
  };

  const selectedClient = selected ? clients.find(c => c.id === selected.id) : null;

  return (
    <div className="app">
      <Topbar name="Administrador" role="Fornecedor" onLogout={onLogout} />
      <div className="main">
        <div className="section-title">Painel do Fornecedor</div>
        <div className="section-sub">Gestão completa de clientes e pagamentos</div>

        <div className="stats-row">
          <div className="stat-box"><div className="stat-val">{clients.length}</div><div className="stat-lbl">Clientes</div></div>
          <div className="stat-box gold"><div className="stat-val">{fmt(totalRevenue)}</div><div className="stat-lbl">Receita total</div></div>
          <div className="stat-box green"><div className="stat-val">{fmt(totalPaid)}</div><div className="stat-lbl">Recebido</div></div>
          <div className="stat-box red"><div className="stat-val">{fmt(totalPending)}</div><div className="stat-lbl">A receber</div></div>
        </div>

        <div className="nav-tabs">
          {[["overview", "Visão Geral"], ["clients", "Clientes"], ["payments", "Todos os Pagamentos"]].map(([v, l]) => (
            <button key={v} className={`nav-tab${tab === v ? " active" : ""}`} onClick={() => setTab(v)}>{l}</button>
          ))}
        </div>

        {/* ── Overview ── */}
        {tab === "overview" && (
          <div>
            <div className="card">
              <div className="section-title" style={{ fontSize: "1.1rem", marginBottom: "1.25rem" }}>Resumo por Cliente</div>
              <div className="tbl-wrap">
                <table>
                  <thead><tr><th>Cliente</th><th>Evento</th><th>Data</th><th>Contrato</th><th>Pago</th><th>Saldo</th><th>Progresso</th></tr></thead>
                  <tbody>
                    {clients.map(c => {
                      const paid = c.payments.filter(p => p.status === "pago").reduce((s, p) => s + p.amount, 0);
                      const pct = c.total > 0 ? Math.round((paid / c.total) * 100) : 0;
                      return (
                        <tr key={c.id} style={{ cursor: "pointer" }} onClick={() => { setSelected(c); setTab("clients"); }}>
                          <td style={{ fontWeight: 500 }}>{c.name}</td>
                          <td style={{ color: "var(--muted)" }}>{c.event}</td>
                          <td style={{ color: "var(--muted)" }}>{fmtDate(c.eventDate)}</td>
                          <td>{fmt(c.total)}</td>
                          <td style={{ color: "var(--sage)" }}>{fmt(paid)}</td>
                          <td style={{ color: paid < c.total ? "var(--rust)" : "var(--sage)" }}>{fmt(c.total - paid)}</td>
                          <td style={{ minWidth: 100 }}>
                            <div className="progress-bar-bg"><div className="progress-bar-fill" style={{ width: pct + "%" }} /></div>
                            <span style={{ fontSize: "0.6rem", color: "var(--muted)" }}>{pct}%</span>
                          </td>
                        </tr>
                      );
                    })}
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        )}

        {/* ── Clients ── */}
        {tab === "clients" && (
          <div>
            <div className="search-row">
              <input className="search-input" placeholder="Buscar cliente, evento ou e-mail…" value={search} onChange={e => setSearch(e.target.value)} />
              <button className="btn btn-gold btn-sm" onClick={() => openModal("addClient", { name: "", email: "", password: "", event: "", eventDate: "", total: "" })}>+ Novo Cliente</button>
            </div>
            <div className="client-grid">
              {filtered.map(c => {
                const paid = c.payments.filter(p => p.status === "pago").reduce((s, p) => s + p.amount, 0);
                const pct = c.total > 0 ? Math.round((paid / c.total) * 100) : 0;
                return (
                  <div key={c.id} className={`client-card${selectedClient?.id === c.id ? " active" : ""}`} onClick={() => setSelected(selectedClient?.id === c.id ? null : c)}>
                    <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start" }}>
                      <div>
                        <div className="client-name">{c.name}</div>
                        <div className="client-meta">{c.event} · {fmtDate(c.eventDate)}</div>
                        <div className="client-meta" style={{ marginTop: "0.2rem" }}>{c.email}</div>
                      </div>
                      <div style={{ display: "flex", gap: "0.3rem" }}>
                        <button className="btn btn-ghost btn-sm" style={{ padding: "0.3rem 0.6rem" }} onClick={e => { e.stopPropagation(); openModal("editClient", { ...c, total: String(c.total) }); }}>✎</button>
                        <button className="btn btn-danger btn-sm" style={{ padding: "0.3rem 0.6rem" }} onClick={e => { e.stopPropagation(); openModal("deleteClient", c); }}>✕</button>
                      </div>
                    </div>
                    <div style={{ marginTop: "0.9rem" }}>
                      <div className="progress-bar-bg"><div className="progress-bar-fill" style={{ width: pct + "%" }} /></div>
                      <div style={{ display: "flex", justifyContent: "space-between", marginTop: "0.3rem" }}>
                        <span style={{ fontSize: "0.65rem", color: "var(--muted)" }}>{fmt(paid)} de {fmt(c.total)}</span>
                        <span style={{ fontSize: "0.65rem", color: "var(--gold)" }}>{pct}%</span>
                      </div>
                    </div>
                  </div>
                );
              })}
              {filtered.length === 0 && <div className="empty-state">Nenhum cliente encontrado.</div>}
            </div>

            {selectedClient && (
              <div className="card" style={{ marginTop: "2rem" }}>
                <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: "1.5rem" }}>
                  <div>
                    <div className="section-title" style={{ fontSize: "1.2rem" }}>{selectedClient.name} — Parcelas</div>
                    <div style={{ fontSize: "0.7rem", color: "var(--muted)" }}>{selectedClient.event}</div>
                  </div>
                  <button className="btn btn-gold btn-sm" onClick={() => openModal("addPayment", { clientId: selectedClient.id, date: "", amount: "", status: "pendente", method: "", note: "" })}>+ Parcela</button>
                </div>
                <div className="tbl-wrap">
                  <table>
                    <thead><tr><th>Vencimento</th><th>Descrição</th><th>Valor</th><th>Método</th><th>Status</th><th>Ações</th></tr></thead>
                    <tbody>
                      {selectedClient.payments.length === 0 && <tr><td colSpan={6} style={{ textAlign: "center", color: "var(--muted)", padding: "2rem" }}>Sem parcelas cadastradas.</td></tr>}
                      {selectedClient.payments.map(p => (
                        <tr key={p.id}>
                          <td>{fmtDate(p.date)}</td>
                          <td>{p.note || "—"}</td>
                          <td style={{ fontWeight: 500 }}>{fmt(p.amount)}</td>
                          <td style={{ color: "var(--muted)" }}>{p.method || "—"}</td>
                          <td><span className={`pill pill-${p.status}`}>{p.status}</span></td>
                          <td>
                            <div className="actions-cell">
                              <button className="btn btn-ghost btn-sm" onClick={() => openModal("editPayment", { ...p, clientId: selectedClient.id, amount: String(p.amount) })}>Editar</button>
                              <button className="btn btn-danger btn-sm" onClick={() => openModal("deletePayment", { ...p, clientId: selectedClient.id })}>Excluir</button>
                            </div>
                          </td>
                        </tr>
                      ))}
                    </tbody>
                  </table>
                </div>
              </div>
            )}
          </div>
        )}

        {/* ── All Payments ── */}
        {tab === "payments" && (
          <div className="card">
            <div className="section-title" style={{ fontSize: "1.1rem", marginBottom: "1.25rem" }}>Todos os Pagamentos</div>
            <div className="search-row">
              <input className="search-input" placeholder="Filtrar por cliente…" value={search} onChange={e => setSearch(e.target.value)} />
              <select className="search-input" style={{ flex: "none", width: 160 }} onChange={e => setSearch(e.target.value)}>
                <option value="">Todos os status</option>
                <option value="pago">Pago</option>
                <option value="pendente">Pendente</option>
              </select>
            </div>
            <div className="tbl-wrap">
              <table>
                <thead><tr><th>Cliente</th><th>Descrição</th><th>Vencimento</th><th>Valor</th><th>Método</th><th>Status</th><th>Ações</th></tr></thead>
                <tbody>
                  {allPayments.filter(p => !search || p.clientName.toLowerCase().includes(search.toLowerCase()) || p.status.includes(search.toLowerCase())).map(p => (
                    <tr key={p.id}>
                      <td style={{ fontWeight: 500 }}>{p.clientName}</td>
                      <td style={{ color: "var(--muted)" }}>{p.note || "—"}</td>
                      <td>{fmtDate(p.date)}</td>
                      <td>{fmt(p.amount)}</td>
                      <td style={{ color: "var(--muted)" }}>{p.method || "—"}</td>
                      <td><span className={`pill pill-${p.status}`}>{p.status}</span></td>
                      <td>
                        <div className="actions-cell">
                          <button className="btn btn-ghost btn-sm" onClick={() => openModal("editPayment", { ...p, amount: String(p.amount) })}>Editar</button>
                          <button className="btn btn-danger btn-sm" onClick={() => openModal("deletePayment", p)}>Excluir</button>
                        </div>
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>
        )}
      </div>

      {/* ── Modals ── */}
      {(modal === "addClient" || modal === "editClient") && (
        <Modal title={modal === "addClient" ? "Novo Cliente" : "Editar Cliente"} onClose={() => setModal(null)}>
          <div className="field"><label>Nome completo *</label><input value={form.name || ""} onChange={e => setForm(f => ({ ...f, name: e.target.value }))} style={{ background: "var(--cream)", borderColor: "#ddd6c8", color: "var(--ink)" }} /></div>
          <div className="field"><label>E-mail *</label><input value={form.email || ""} onChange={e => setForm(f => ({ ...f, email: e.target.value }))} style={{ background: "var(--cream)", borderColor: "#ddd6c8", color: "var(--ink)" }} /></div>
          {modal === "addClient" && <div className="field"><label>Senha inicial</label><input value={form.password || ""} onChange={e => setForm(f => ({ ...f, password: e.target.value }))} placeholder="senha123" style={{ background: "var(--cream)", borderColor: "#ddd6c8", color: "var(--ink)" }} /></div>}
          <div className="field"><label>Evento *</label><input value={form.event || ""} onChange={e => setForm(f => ({ ...f, event: e.target.value }))} style={{ background: "var(--cream)", borderColor: "#ddd6c8", color: "var(--ink)" }} /></div>
          <div className="field"><label>Data do evento</label><input type="date" value={form.eventDate || ""} onChange={e => setForm(f => ({ ...f, eventDate: e.target.value }))} style={{ background: "var(--cream)", borderColor: "#ddd6c8", color: "var(--ink)" }} /></div>
          <div className="field"><label>Valor total (R$)</label><input type="number" value={form.total || ""} onChange={e => setForm(f => ({ ...f, total: e.target.value }))} style={{ background: "var(--cream)", borderColor: "#ddd6c8", color: "var(--ink)" }} /></div>
          <div className="modal-footer">
            <button className="btn btn-ghost" onClick={() => setModal(null)}>Cancelar</button>
            <button className="btn btn-gold btn-sm" style={{ width: "auto" }} onClick={saveClient}>Salvar</button>
          </div>
        </Modal>
      )}

      {modal === "deleteClient" && (
        <Modal title="Excluir Cliente" onClose={() => setModal(null)}>
          <p style={{ fontSize: "0.85rem", color: "var(--muted)", lineHeight: 1.6 }}>Deseja excluir <strong style={{ color: "var(--ink)" }}>{form.name}</strong> e todos os seus pagamentos? Esta ação é irreversível.</p>
          <div className="modal-footer">
            <button className="btn btn-ghost" onClick={() => setModal(null)}>Cancelar</button>
            <button className="btn btn-danger" onClick={deleteClient}>Excluir</button>
          </div>
        </Modal>
      )}

      {(modal === "addPayment" || modal === "editPayment") && (
        <Modal title={modal === "addPayment" ? "Nova Parcela" : "Editar Parcela"} onClose={() => setModal(null)}>
          <div className="field"><label>Vencimento *</label><input type="date" value={form.date || ""} onChange={e => setForm(f => ({ ...f, date: e.target.value }))} style={{ background: "var(--cream)", borderColor: "#ddd6c8", color: "var(--ink)" }} /></div>
          <div className="field"><label>Valor (R$) *</label><input type="number" value={form.amount || ""} onChange={e => setForm(f => ({ ...f, amount: e.target.value }))} style={{ background: "var(--cream)", borderColor: "#ddd6c8", color: "var(--ink)" }} /></div>
          <div className="field"><label>Descrição</label><input value={form.note || ""} onChange={e => setForm(f => ({ ...f, note: e.target.value }))} style={{ background: "var(--cream)", borderColor: "#ddd6c8", color: "var(--ink)" }} /></div>
          <div className="field"><label>Status</label>
            <select value={form.status || "pendente"} onChange={e => setForm(f => ({ ...f, status: e.target.value }))} style={{ background: "var(--cream)", borderColor: "#ddd6c8", color: "var(--ink)" }}>
              <option value="pendente">Pendente</option>
              <option value="pago">Pago</option>
              <option value="atrasado">Atrasado</option>
            </select>
          </div>
          <div className="field"><label>Forma de pagamento</label>
            <select value={form.method || ""} onChange={e => setForm(f => ({ ...f, method: e.target.value }))} style={{ background: "var(--cream)", borderColor: "#ddd6c8", color: "var(--ink)" }}>
              <option value="">—</option>
              {["PIX", "Transferência", "Cartão de Crédito", "Cartão de Débito", "Dinheiro", "Boleto"].map(m => <option key={m}>{m}</option>)}
            </select>
          </div>
          <div className="modal-footer">
            <button className="btn btn-ghost" onClick={() => setModal(null)}>Cancelar</button>
            <button className="btn btn-gold btn-sm" style={{ width: "auto" }} onClick={savePayment}>Salvar</button>
          </div>
        </Modal>
      )}

      {modal === "deletePayment" && (
        <Modal title="Excluir Parcela" onClose={() => setModal(null)}>
          <p style={{ fontSize: "0.85rem", color: "var(--muted)", lineHeight: 1.6 }}>Excluir a parcela <strong style={{ color: "var(--ink)" }}>{form.note || fmt(form.amount)}</strong>? Esta ação é irreversível.</p>
          <div className="modal-footer">
            <button className="btn btn-ghost" onClick={() => setModal(null)}>Cancelar</button>
            <button className="btn btn-danger" onClick={deletePayment}>Excluir</button>
          </div>
        </Modal>
      )}
    </div>
  );
}

// ─── Shared Components ────────────────────────────────────────────────────────
function Topbar({ name, role, onLogout }) {
  return (
    <div className="topbar">
      <div className="topbar-brand">Buffet · Pagamentos</div>
      <div className="topbar-right">
        <span className="topbar-user">{name}</span>
        <span className="badge-role">{role}</span>
        <button className="btn btn-ghost" onClick={onLogout}>Sair</button>
      </div>
    </div>
  );
}

function Modal({ title, onClose, children }) {
  return (
    <div className="modal-overlay" onClick={e => e.target === e.currentTarget && onClose()}>
      <div className="modal">
        <div className="modal-title">{title}</div>
        {children}
      </div>
    </div>
  );
}

function fmtDate(str) {
  if (!str) return "—";
  const [y, m, d] = str.split("-");
  return `${d}/${m}/${y}`;
}
