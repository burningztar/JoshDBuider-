

// ─── Mock Data Generators ───────────────────────────────────────────────────
const SOL_TOKENS = [
  { name: "BONK", symbol: "BONK", chain: "SOL" },
  { name: "dogwifhat", symbol: "WIF", chain: "SOL" },
  { name: "Popcat", symbol: "POPCAT", chain: "SOL" },
  { name: "Moo Deng", symbol: "MOODENG", chain: "SOL" },
  { name: "GOAT", symbol: "GOAT", chain: "SOL" },
  { name: "Fartcoin", symbol: "FART", chain: "SOL" },
  { name: "Gigachad", symbol: "GIGA", chain: "SOL" },
  { name: "ai16z", symbol: "AI16Z", chain: "SOL" },
];
const BNB_TOKENS = [
  { name: "SafeMoon", symbol: "SAFEMOON", chain: "BNB" },
  { name: "BabyDoge", symbol: "BABYDOGE", chain: "BNB" },
  { name: "Floki", symbol: "FLOKI", chain: "BNB" },
  { name: "PancakeSwap", symbol: "CAKE", chain: "BNB" },
  { name: "CatGirl", symbol: "CATGIRL", chain: "BNB" },
  { name: "BurnDoge", symbol: "BDOGE", chain: "BNB" },
  { name: "ApeSwap", symbol: "BANANA", chain: "BNB" },
  { name: "Kishu Inu", symbol: "KISHU", chain: "BNB" },
];

const rnd = (min, max) => Math.random() * (max - min) + min;
const randInt = (min, max) => Math.floor(rnd(min, max));

function generateToken(base) {
  const pumpScore = rnd(0, 100);
  const volumeSpike = rnd(1, 50);
  const walletAge = randInt(1, 120);
  const holders = randInt(50, 50000);
  const marketCap = rnd(10000, 50000000);
  const change1h = rnd(-30, 150);
  const change24h = rnd(-60, 400);
  const liquidity = rnd(5000, 5000000);
  const devWallet = rnd(0, 30);
  const isEarlyMover = pumpScore > 72 && volumeSpike > 15 && walletAge < 48;
  const riskLevel = devWallet > 20 ? "HIGH" : devWallet > 10 ? "MED" : "LOW";
  const signal =
    pumpScore > 80 ? "🚀 STRONG BUY" :
    pumpScore > 60 ? "📈 ACCUMULATE" :
    pumpScore > 40 ? "👀 WATCH" :
    "⚠️ CAUTION";
  return {
    ...base,
    id: `${base.symbol}-${Date.now()}-${Math.random()}`,
    pumpScore: Math.round(pumpScore),
    volumeSpike: volumeSpike.toFixed(1),
    walletAge,
    holders,
    marketCap,
    change1h: change1h.toFixed(2),
    change24h: change24h.toFixed(2),
    liquidity,
    devWallet: devWallet.toFixed(1),
    isEarlyMover,
    riskLevel,
    signal,
    timestamp: new Date().toISOString(),
    txCount: randInt(100, 50000),
    newWallets: randInt(10, 5000),
    price: rnd(0.000001, 0.5).toFixed(8),
    priceHistory: Array.from({ length: 24 }, (_, i) => ({
      t: i,
      v: rnd(0.5, 5) * (1 + i * 0.05 * (Math.random() - 0.3))
    })),
  };
}

function generateLiveTokens() {
  return [...SOL_TOKENS, ...BNB_TOKENS].map(generateToken);
}

function generateAlerts(tokens) {
  return tokens
    .filter(t => t.isEarlyMover)
    .slice(0, 5)
    .map(t => ({
      id: Math.random(),
      token: t,
      message: `Early mover detected on ${t.chain}: ${t.name} (${t.symbol}) — ${t.change1h}% in 1h, Vol spike ${t.volumeSpike}x`,
      time: new Date().toLocaleTimeString(),
      type: t.pumpScore > 80 ? "critical" : "warning",
    }));
}

// ─── Sparkline ───────────────────────────────────────────────────────────────
function Sparkline({ data, color }) {
  const w = 80, h = 28;
  if (!data || data.length < 2) return null;
  const vals = data.map(d => d.v);
  const min = Math.min(...vals), max = Math.max(...vals);
  const norm = v => h - ((v - min) / (max - min || 1)) * h;
  const pts = data.map((d, i) => `${(i / (data.length - 1)) * w},${norm(d.v)}`).join(" ");
  return (
    <svg width={w} height={h} style={{ display: "block" }}>
      <polyline points={pts} fill="none" stroke={color} strokeWidth="1.5" strokeLinecap="round" />
    </svg>
  );
}

// ─── Mini Bar ────────────────────────────────────────────────────────────────
function ScoreBar({ value, max = 100, color }) {
  return (
    <div style={{ display: "flex", alignItems: "center", gap: 6 }}>
      <div style={{ flex: 1, height: 5, background: "#1a1f2e", borderRadius: 3 }}>
        <div style={{ width: `${(value / max) * 100}%`, height: "100%", background: color, borderRadius: 3, transition: "width 0.4s" }} />
      </div>
      <span style={{ fontSize: 11, color, fontFamily: "monospace", minWidth: 28 }}>{value}</span>
    </div>
  );
}

// ─── AI Analysis Panel ───────────────────────────────────────────────────────
function AIAnalysis({ tokens, patterns }) {
  const [analysis, setAnalysis] = useState("");
  const [loading, setLoading] = useState(false);
  const [mode, setMode] = useState("market");

  const prompts = {
    market: `You are an elite crypto market analyst specialized in memecoin early detection on Solana and BNB chains.

Here is LIVE market data for ${tokens.length} tokens:
${JSON.stringify(tokens.slice(0,8).map(t => ({
  name: t.name, chain: t.chain, pumpScore: t.pumpScore,
  change1h: t.change1h + "%", change24h: t.change24h + "%",
  volumeSpike: t.volumeSpike + "x", holders: t.holders,
  marketCap: "$" + Math.round(t.marketCap).toLocaleString(),
  devWallet: t.devWallet + "%", riskLevel: t.riskLevel,
  signal: t.signal, newWallets: t.newWallets,
})), null, 2)}

Analyze:
1. 🔥 Top 3 early movers with highest conviction (explain WHY)
2. 📊 Market trend pattern — is this a pump cycle, accumulation, or distribution phase?
3. ⚠️ Risk flags — rug indicators, dev wallet concentrations
4. 🎯 Actionable strategy for the next 2-4 hours

Be specific, data-driven, and direct. No generic advice. Format with emojis and clear sections.`,

    pattern: `You are a pattern recognition AI for crypto memecoin markets on Solana and BNB chains.

Current scan data:
- Early movers detected: ${tokens.filter(t => t.isEarlyMover).length}
- Avg pump score: ${Math.round(tokens.reduce((a, t) => a + t.pumpScore, 0) / tokens.length)}
- High-risk tokens: ${tokens.filter(t => t.riskLevel === "HIGH").length}
- SOL chain activity: ${tokens.filter(t => t.chain === "SOL" && parseFloat(t.change1h) > 20).length} tokens up >20% in 1h
- BNB chain activity: ${tokens.filter(t => t.chain === "BNB" && parseFloat(t.change1h) > 20).length} tokens up >20% in 1h

Historical patterns noted: ${patterns.join(", ") || "Accumulating data..."}

Identify:
1. 🧠 The dominant market pattern right now (e.g., rotation, narrative play, whale accumulation)
2. 📈 Which chain (SOL vs BNB) is showing stronger momentum and why
3. 🔄 Cycle phase prediction — are we early, mid, or late in the pump cycle?
4. 💡 Pattern-based entry/exit signals with specific price action triggers
5. 🕵️ Whale behavior indicators from new wallet counts and volume spikes

Give sharp, actionable intel. Be a street-smart analyst, not a textbook.`,

    risk: `You are a crypto risk intelligence analyst focusing on memecoin rugs, honeypots, and scams on Solana and BNB.

Tokens to risk-assess:
${JSON.stringify(tokens.slice(0,6).map(t => ({
  name: t.name, chain: t.chain,
  devWallet: t.devWallet + "%",
  walletAge: t.walletAge + " hours",
  holders: t.holders,
  liquidity: "$" + Math.round(t.liquidity).toLocaleString(),
  txCount: t.txCount,
  riskLevel: t.riskLevel,
})), null, 2)}

Provide:
1. 🚨 Red flag tokens — rank from most to least dangerous
2. 🛡️ Safe plays — tokens with strongest fundamentals
3. 📋 Rug indicators checklist — what to look for before buying
4. 💰 Liquidity traps — which tokens may have locked/fake liquidity
5. ⚡ Quick safety checks you can do on-chain before entering

Be brutally honest. Protect the trader.`,
  };

  const runAnalysis = async () => {
    setLoading(true);
    setAnalysis("");
    try {
      const res = await fetch(CLAUDE_API, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514",
          max_tokens: 1000,
          messages: [{ role: "user", content: prompts[mode] }],
        }),
      });
      const data = await res.json();
      setAnalysis(data.content?.[0]?.text || "Analysis unavailable.");
    } catch (e) {
      setAnalysis("⚠️ API connection failed. Check your network.");
    }
    setLoading(false);
  };

  return (
    <div style={{ background: "#0d1117", border: "1px solid #1e2a3a", borderRadius: 12, padding: 20, height: "100%" }}>
      <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 16 }}>
        <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
          <span style={{ fontSize: 18 }}>🤖</span>
          <span style={{ color: "#e2e8f0", fontWeight: 700, fontSize: 15, fontFamily: "'Space Mono', monospace" }}>AI INTELLIGENCE</span>
        </div>
        <div style={{ display: "flex", gap: 4 }}>
          {["market", "pattern", "risk"].map(m => (
            <button key={m} onClick={() => setMode(m)} style={{
              padding: "4px 10px", borderRadius: 6, border: "none", cursor: "pointer", fontSize: 11,
              fontFamily: "'Space Mono', monospace", fontWeight: 600, textTransform: "uppercase",
              background: mode === m ? "#00d4ff" : "#1a2332",
              color: mode === m ? "#000" : "#64748b",
              transition: "all 0.2s",
            }}>{m}</button>
          ))}
        </div>
      </div>

      <button onClick={runAnalysis} disabled={loading} style={{
        width: "100%", padding: "10px 0", borderRadius: 8, border: "none", cursor: loading ? "not-allowed" : "pointer",
        background: loading ? "#1a2332" : "linear-gradient(135deg, #00d4ff, #7b2ff7)",
        color: "#fff", fontWeight: 700, fontSize: 13, fontFamily: "'Space Mono', monospace",
        marginBottom: 14, transition: "all 0.3s",
        boxShadow: loading ? "none" : "0 0 20px rgba(0,212,255,0.3)",
      }}>
        {loading ? "⚡ ANALYZING MARKET..." : `▶ RUN ${mode.toUpperCase()} ANALYSIS`}
      </button>

      <div style={{
        background: "#060a0f", borderRadius: 8, padding: 14, minHeight: 220, maxHeight: 340,
        overflowY: "auto", border: "1px solid #0f1923",
        fontFamily: "'Space Mono', monospace", fontSize: 12, color: "#94a3b8", lineHeight: 1.7,
        whiteSpace: "pre-wrap",
      }}>
        {loading ? (
          <div style={{ textAlign: "center", paddingTop: 60 }}>
            <div style={{ color: "#00d4ff", fontSize: 24, animation: "pulse 1s infinite" }}>⟳</div>
            <div style={{ color: "#4a5568", marginTop: 8 }}>Scanning {tokens.length} tokens across SOL + BNB...</div>
          </div>
        ) : analysis ? (
          <span style={{ color: "#e2e8f0" }}>{analysis}</span>
        ) : (
          <div style={{ color: "#2d3748", textAlign: "center", paddingTop: 60 }}>
            Click RUN ANALYSIS to get AI-powered market intelligence
          </div>
        )}
      </div>
    </div>
  );
}

// ─── Token Row ────────────────────────────────────────────────────────────────
function TokenRow({ token, rank }) {
  const isUp1h = parseFloat(token.change1h) > 0;
  const isUp24h = parseFloat(token.change24h) > 0;
  const chainColor = token.chain === "SOL" ? "#9945ff" : "#f3ba2f";
  const scoreColor = token.pumpScore > 70 ? "#00ff88" : token.pumpScore > 45 ? "#ffd700" : "#ff4757";

  return (
    <div style={{
      display: "grid", gridTemplateColumns: "28px 90px 60px 1fr 80px 80px 70px 70px 80px 80px",
      gap: 0, alignItems: "center", padding: "8px 14px",
      background: token.isEarlyMover ? "rgba(0,255,136,0.04)" : "transparent",
      borderBottom: "1px solid #0f1923",
      borderLeft: token.isEarlyMover ? "2px solid #00ff88" : "2px solid transparent",
      transition: "background 0.2s",
      cursor: "default",
    }}
      onMouseEnter={e => e.currentTarget.style.background = "rgba(255,255,255,0.03)"}
      onMouseLeave={e => e.currentTarget.style.background = token.isEarlyMover ? "rgba(0,255,136,0.04)" : "transparent"}
    >
      <span style={{ color: "#2d3748", fontSize: 11, fontFamily: "monospace" }}>#{rank}</span>
      <div>
        <div style={{ display: "flex", alignItems: "center", gap: 5 }}>
          <span style={{ fontSize: 9, background: chainColor, color: "#000", borderRadius: 3, padding: "1px 4px", fontWeight: 800, fontFamily: "monospace" }}>{token.chain}</span>
          <span style={{ color: "#e2e8f0", fontSize: 12, fontWeight: 700 }}>{token.symbol}</span>
          {token.isEarlyMover && <span style={{ fontSize: 9 }}>⚡</span>}
        </div>
        <div style={{ color: "#4a5568", fontSize: 10 }}>{token.name}</div>
      </div>
      <span style={{ color: "#64748b", fontSize: 11, fontFamily: "monospace" }}>${token.price}</span>
      <div style={{ padding: "0 8px" }}>
        <Sparkline data={token.priceHistory} color={isUp1h ? "#00ff88" : "#ff4757"} />
      </div>
      <span style={{ color: isUp1h ? "#00ff88" : "#ff4757", fontSize: 12, fontFamily: "monospace", textAlign: "right" }}>
        {isUp1h ? "+" : ""}{token.change1h}%
      </span>
      <span style={{ color: isUp24h ? "#00ff88" : "#ff4757", fontSize: 12, fontFamily: "monospace", textAlign: "right" }}>
        {isUp24h ? "+" : ""}{token.change24h}%
      </span>
      <div style={{ padding: "0 4px" }}>
        <ScoreBar value={token.pumpScore} color={scoreColor} />
      </div>
      <div style={{
        fontSize: 10, fontWeight: 700, textAlign: "center", padding: "2px 6px", borderRadius: 4,
        background: token.riskLevel === "HIGH" ? "rgba(255,71,87,0.15)" : token.riskLevel === "MED" ? "rgba(255,215,0,0.15)" : "rgba(0,255,136,0.1)",
        color: token.riskLevel === "HIGH" ? "#ff4757" : token.riskLevel === "MED" ? "#ffd700" : "#00ff88",
        fontFamily: "monospace",
      }}>{token.riskLevel}</div>
      <div style={{ fontSize: 10, color: "#94a3b8", textAlign: "right", fontFamily: "monospace" }}>{token.signal.split(" ")[0]}</div>
    </div>
  );
}

// ─── Stats Card ───────────────────────────────────────────────────────────────
function StatCard({ label, value, sub, color, icon }) {
  return (
    <div style={{
      background: "#0d1117", border: "1px solid #1e2a3a", borderRadius: 10, padding: "14px 16px",
      borderTop: `2px solid ${color}`,
    }}>
      <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start" }}>
        <div>
          <div style={{ color: "#4a5568", fontSize: 10, fontFamily: "'Space Mono', monospace", textTransform: "uppercase", letterSpacing: 1, marginBottom: 4 }}>{label}</div>
          <div style={{ color, fontSize: 22, fontWeight: 800, fontFamily: "'Space Mono', monospace" }}>{value}</div>
          {sub && <div style={{ color: "#2d3748", fontSize: 10, marginTop: 2 }}>{sub}</div>}
        </div>
        <span style={{ fontSize: 22 }}>{icon}</span>
      </div>
    </div>
  );
}

// ─── Alert Feed ───────────────────────────────────────────────────────────────
function AlertFeed({ alerts }) {
  return (
    <div style={{ background: "#0d1117", border: "1px solid #1e2a3a", borderRadius: 12, padding: 16 }}>
      <div style={{ display: "flex", alignItems: "center", gap: 8, marginBottom: 12 }}>
        <span style={{ width: 8, height: 8, borderRadius: "50%", background: "#ff4757", display: "inline-block", boxShadow: "0 0 8px #ff4757", animation: "pulse 1s infinite" }} />
        <span style={{ color: "#e2e8f0", fontWeight: 700, fontSize: 13, fontFamily: "'Space Mono', monospace" }}>LIVE ALERTS</span>
      </div>
      <div style={{ display: "flex", flexDirection: "column", gap: 8, maxHeight: 200, overflowY: "auto" }}>
        {alerts.length === 0 ? (
          <div style={{ color: "#2d3748", fontSize: 12, textAlign: "center", padding: 20, fontFamily: "monospace" }}>Scanning for early movers...</div>
        ) : alerts.map(a => (
          <div key={a.id} style={{
            background: a.type === "critical" ? "rgba(255,71,87,0.08)" : "rgba(255,215,0,0.06)",
            border: `1px solid ${a.type === "critical" ? "rgba(255,71,87,0.3)" : "rgba(255,215,0,0.2)"}`,
            borderRadius: 8, padding: "8px 12px",
          }}>
            <div style={{ color: a.type === "critical" ? "#ff4757" : "#ffd700", fontSize: 11, fontFamily: "monospace", marginBottom: 2 }}>
              {a.type === "critical" ? "🚨" : "⚠️"} {a.time}
            </div>
            <div style={{ color: "#94a3b8", fontSize: 11, lineHeight: 1.5 }}>{a.message}</div>
          </div>
        ))}
      </div>
    </div>
  );
}

// ─── Main App ─────────────────────────────────────────────────────────────────
export default function App() {
  const [tokens, setTokens] = useState([]);
  const [alerts, setAlerts] = useState([]);
  const [patterns, setPatterns] = useState([]);
  const [filter, setFilter] = useState("ALL");
  const [sortBy, setSortBy] = useState("pumpScore");
  const [scanCount, setScanCount] = useState(0);
  const [lastScan, setLastScan] = useState("—");
  const [scanning, setScanning] = useState(false);
  const intervalRef = useRef(null);

  const scan = useCallback(() => {
    setScanning(true);
    setTimeout(() => {
      const newTokens = generateLiveTokens();
      setTokens(newTokens);
      setAlerts(prev => [...generateAlerts(newTokens), ...prev].slice(0, 20));
      setScanCount(c => c + 1);
      setLastScan(new Date().toLocaleTimeString());
      setScanning(false);

      // pattern accumulation
      const solMomentum = newTokens.filter(t => t.chain === "SOL" && parseFloat(t.change1h) > 20).length;
      const bnbMomentum = newTokens.filter(t => t.chain === "BNB" && parseFloat(t.change1h) > 20).length;
      if (solMomentum > 2) setPatterns(p => [...new Set([...p, "SOL rotation active"])].slice(-8));
      if (bnbMomentum > 2) setPatterns(p => [...new Set([...p, "BNB volume surge"])].slice(-8));
      const avgScore = newTokens.reduce((a, t) => a + t.pumpScore, 0) / newTokens.length;
      if (avgScore > 55) setPatterns(p => [...new Set([...p, "Bull cycle detected"])].slice(-8));
    }, 600);
  }, []);

  useEffect(() => {
    scan();
    intervalRef.current = setInterval(scan, 15000);
    return () => clearInterval(intervalRef.current);
  }, [scan]);

  const displayed = tokens
    .filter(t => filter === "ALL" ? true : filter === "EARLY" ? t.isEarlyMover : t.chain === filter)
    .sort((a, b) => {
      if (sortBy === "pumpScore") return b.pumpScore - a.pumpScore;
      if (sortBy === "change1h") return parseFloat(b.change1h) - parseFloat(a.change1h);
      if (sortBy === "change24h") return parseFloat(b.change24h) - parseFloat(a.change24h);
      return 0;
    });

  const earlyMovers = tokens.filter(t => t.isEarlyMover).length;
  const avgScore = tokens.length ? Math.round(tokens.reduce((a, t) => a + t.pumpScore, 0) / tokens.length) : 0;
  const highRisk = tokens.filter(t => t.riskLevel === "HIGH").length;
  const solCount = tokens.filter(t => t.chain === "SOL" && parseFloat(t.change1h) > 10).length;

  return (
    <div style={{
      minHeight: "100vh", background: "#060a0f", fontFamily: "'Space Mono', monospace",
      color: "#e2e8f0",
    }}>
      <link href="https://fonts.googleapis.com/css2?family=Space+Mono:ital,wght@0,400;0,700;1,400&display=swap" rel="stylesheet" />
      <style>{`
        * { box-sizing: border-box; margin: 0; padding: 0; }
        ::-webkit-scrollbar { width: 4px; height: 4px; }
        ::-webkit-scrollbar-track { background: #060a0f; }
        ::-webkit-scrollbar-thumb { background: #1e2a3a; border-radius: 2px; }
        @keyframes pulse { 0%,100%{opacity:1} 50%{opacity:0.3} }
        @keyframes slideIn { from{transform:translateY(-8px);opacity:0} to{transform:translateY(0);opacity:1} }
        @keyframes scanline { 0%{top:-4px} 100%{top:100%} }
      `}</style>

      {/* Header */}
      <div style={{
        borderBottom: "1px solid #1e2a3a", padding: "14px 24px",
        display: "flex", alignItems: "center", justifyContent: "space-between",
        background: "rgba(6,10,15,0.95)", position: "sticky", top: 0, zIndex: 100,
        backdropFilter: "blur(10px)",
      }}>
        <div style={{ display: "flex", alignItems: "center", gap: 16 }}>
          <div style={{ position: "relative" }}>
            <div style={{ fontSize: 22, lineHeight: 1 }}>⚡</div>
          </div>
          <div>
            <div style={{ fontSize: 16, fontWeight: 700, color: "#00d4ff", letterSpacing: 2 }}>MEMESCOUT AI</div>
            <div style={{ fontSize: 10, color: "#2d3748", letterSpacing: 1 }}>SOL + BNB EARLY MOVER DETECTION</div>
          </div>
        </div>

        <div style={{ display: "flex", alignItems: "center", gap: 16 }}>
          {patterns.slice(-3).map((p, i) => (
            <span key={i} style={{
              fontSize: 10, background: "rgba(0,212,255,0.1)", border: "1px solid rgba(0,212,255,0.2)",
              color: "#00d4ff", padding: "3px 8px", borderRadius: 4,
            }}>📊 {p}</span>
          ))}
          <div style={{ textAlign: "right" }}>
            <div style={{ fontSize: 10, color: "#2d3748" }}>LAST SCAN</div>
            <div style={{ fontSize: 12, color: "#94a3b8" }}>{lastScan}</div>
          </div>
          <div style={{
            display: "flex", alignItems: "center", gap: 6, background: "#0d1117",
            border: "1px solid #1e2a3a", borderRadius: 8, padding: "6px 12px",
          }}>
            <span style={{
              width: 7, height: 7, borderRadius: "50%", display: "inline-block",
              background: scanning ? "#ffd700" : "#00ff88",
              boxShadow: `0 0 8px ${scanning ? "#ffd700" : "#00ff88"}`,
              animation: "pulse 1s infinite",
            }} />
            <span style={{ fontSize: 11, color: scanning ? "#ffd700" : "#00ff88" }}>
              {scanning ? "SCANNING" : "LIVE"} #{scanCount}
            </span>
          </div>
        </div>
      </div>

      <div style={{ padding: "20px 24px", maxWidth: 1600, margin: "0 auto" }}>

        {/* Stats Row */}
        <div style={{ display: "grid", gridTemplateColumns: "repeat(5, 1fr)", gap: 12, marginBottom: 20 }}>
          <StatCard label="Early Movers" value={earlyMovers} sub="⚡ high conviction" color="#00ff88" icon="🚀" />
          <StatCard label="Avg Pump Score" value={avgScore} sub="across all tokens" color="#00d4ff" icon="📊" />
          <StatCard label="High Risk" value={highRisk} sub="⚠️ rug indicators" color="#ff4757" icon="🚨" />
          <StatCard label="SOL Hot" value={`${solCount}`} sub=">10% in 1h" color="#9945ff" icon="◎" />
          <StatCard label="Total Tracked" value={tokens.length} sub="SOL + BNB tokens" color="#ffd700" icon="👁" />
        </div>

        {/* Main Grid */}
        <div style={{ display: "grid", gridTemplateColumns: "1fr 380px", gap: 16, marginBottom: 16 }}>

          {/* Token Table */}
          <div style={{ background: "#0d1117", border: "1px solid #1e2a3a", borderRadius: 12, overflow: "hidden" }}>
            {/* Table Controls */}
            <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", padding: "12px 14px", borderBottom: "1px solid #0f1923" }}>
              <div style={{ display: "flex", gap: 6 }}>
                {["ALL", "EARLY", "SOL", "BNB"].map(f => (
                  <button key={f} onClick={() => setFilter(f)} style={{
                    padding: "4px 10px", borderRadius: 6, border: "none", cursor: "pointer",
                    fontSize: 10, fontFamily: "'Space Mono', monospace", fontWeight: 700,
                    background: filter === f ? (f === "SOL" ? "#9945ff" : f === "BNB" ? "#f3ba2f" : f === "EARLY" ? "#00ff88" : "#00d4ff") : "#1a2332",
                    color: filter === f ? "#000" : "#4a5568",
                  }}>{f === "EARLY" ? "⚡ EARLY" : f}</button>
                ))}
              </div>
              <div style={{ display: "flex", gap: 6, alignItems: "center" }}>
                <span style={{ fontSize: 10, color: "#2d3748" }}>SORT:</span>
                {["pumpScore", "change1h", "change24h"].map(s => (
                  <button key={s} onClick={() => setSortBy(s)} style={{
                    padding: "3px 8px", borderRadius: 4, border: "none", cursor: "pointer",
                    fontSize: 9, fontFamily: "'Space Mono', monospace",
                    background: sortBy === s ? "#00d4ff" : "#1a2332",
                    color: sortBy === s ? "#000" : "#4a5568",
                  }}>{s === "pumpScore" ? "SCORE" : s === "change1h" ? "1H %" : "24H %"}</button>
                ))}
              </div>
            </div>

            {/* Table Header */}
            <div style={{
              display: "grid", gridTemplateColumns: "28px 90px 60px 1fr 80px 80px 70px 70px 80px 80px",
              gap: 0, padding: "6px 14px", background: "#060a0f",
            }}>
              {["#", "TOKEN", "PRICE", "CHART", "1H %", "24H %", "SCORE", "RISK", "SIGNAL"].map((h, i) => (
                <span key={i} style={{ fontSize: 9, color: "#2d3748", letterSpacing: 1, textTransform: "uppercase" }}>{h}</span>
              ))}
            </div>

            {/* Token Rows */}
            <div style={{ maxHeight: 420, overflowY: "auto" }}>
              {displayed.map((token, i) => (
                <TokenRow key={token.id} token={token} rank={i + 1} />
              ))}
            </div>
          </div>

          {/* Right Column */}
          <div style={{ display: "flex", flexDirection: "column", gap: 14 }}>
            <AlertFeed alerts={alerts} />

            {/* Chain Comparison */}
            <div style={{ background: "#0d1117", border: "1px solid #1e2a3a", borderRadius: 12, padding: 16 }}>
              <div style={{ color: "#e2e8f0", fontWeight: 700, fontSize: 13, marginBottom: 12 }}>⚔️ CHAIN BATTLE</div>
              {["SOL", "BNB"].map(chain => {
                const chainTokens = tokens.filter(t => t.chain === chain);
                const avg = chainTokens.length ? Math.round(chainTokens.reduce((a, t) => a + t.pumpScore, 0) / chainTokens.length) : 0;
                const movers = chainTokens.filter(t => t.isEarlyMover).length;
                const color = chain === "SOL" ? "#9945ff" : "#f3ba2f";
                return (
                  <div key={chain} style={{ marginBottom: 14 }}>
                    <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 6 }}>
                      <span style={{ color, fontWeight: 700, fontSize: 13 }}>{chain === "SOL" ? "◎ Solana" : "⬡ BNB Chain"}</span>
                      <span style={{ color: "#94a3b8", fontSize: 11 }}>{movers} early movers</span>
                    </div>
                    <ScoreBar value={avg} color={color} />
                    <div style={{ display: "flex", gap: 10, marginTop: 6 }}>
                      <span style={{ fontSize: 10, color: "#4a5568" }}>Avg Score: <span style={{ color }}>{avg}</span></span>
                      <span style={{ fontSize: 10, color: "#4a5568" }}>Hot: <span style={{ color }}>{chainTokens.filter(t => parseFloat(t.change1h) > 20).length}</span></span>
                    </div>
                  </div>
                );
              })}
            </div>

            {/* Top Early Movers Quick View */}
            <div style={{ background: "#0d1117", border: "1px solid rgba(0,255,136,0.2)", borderRadius: 12, padding: 16 }}>
              <div style={{ color: "#00ff88", fontWeight: 700, fontSize: 13, marginBottom: 12 }}>⚡ TOP EARLY MOVERS</div>
              {tokens.filter(t => t.isEarlyMover).slice(0, 4).map(t => (
                <div key={t.id} style={{
                  display: "flex", justifyContent: "space-between", alignItems: "center",
                  padding: "6px 0", borderBottom: "1px solid #0f1923",
                }}>
                  <div>
                    <span style={{ fontSize: 11, fontWeight: 700, color: "#e2e8f0" }}>{t.symbol}</span>
                    <span style={{ fontSize: 9, color: t.chain === "SOL" ? "#9945ff" : "#f3ba2f", marginLeft: 5 }}>{t.chain}</span>
                  </div>
                  <div style={{ textAlign: "right" }}>
                    <div style={{ fontSize: 11, color: "#00ff88", fontFamily: "monospace" }}>+{t.change1h}%</div>
                    <div style={{ fontSize: 9, color: "#4a5568" }}>{t.volumeSpike}x vol</div>
                  </div>
                </div>
              ))}
              {tokens.filter(t => t.isEarlyMover).length === 0 && (
                <div style={{ color: "#2d3748", fontSize: 11, textAlign: "center", padding: 16 }}>No early movers in current scan</div>
              )}
            </div>
          </div>
        </div>

        {/* AI Panel */}
        <AIAnalysis tokens={tokens} patterns={patterns} />

        {/* Footer */}
        <div style={{ marginTop: 20, textAlign: "center", color: "#1e2a3a", fontSize: 10, letterSpacing: 1 }}>
          MEMESCOUT AI • NOT FINANCIAL ADVICE • DYOR • SOL + BNB CHAIN MONITOR
        </div>
      </div>
    </div>
  );
}
