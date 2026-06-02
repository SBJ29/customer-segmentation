import { useState, useEffect, useRef } from "react";

const COLORS = {
  champions:    { bg: "#1D9E75", light: "#E1F5EE", text: "#085041", border: "#5DCAA5" },
  loyal:        { bg: "#378ADD", light: "#E6F1FB", text: "#0C447C", border: "#85B7EB" },
  potential:    { bg: "#EF9F27", light: "#FAEEDA", text: "#633806", border: "#FAC775" },
  atrisk:       { bg: "#E24B4A", light: "#FCEBEB", text: "#791F1F", border: "#F09595" },
  newcustomers: { bg: "#7F77DD", light: "#EEEDFE", text: "#3C3489", border: "#AFA9EC" },
};

const SEGMENTS = [
  {
    id: "champions",
    label: "Champions",
    icon: "🏆",
    count: 842,
    pct: 22,
    avgSpend: 4820,
    frequency: 8.4,
    recency: 6,
    clv: 32400,
    desc: "High spend, high frequency, recent buyers. Your most valuable customers.",
    traits: ["Loyal brand advocates", "Early adopters", "High AOV"],
    actions: ["Reward with VIP perks", "Ask for reviews", "Upsell premium"],
  },
  {
    id: "loyal",
    label: "Loyal Customers",
    icon: "⭐",
    count: 1124,
    pct: 29,
    avgSpend: 2340,
    frequency: 5.1,
    recency: 18,
    clv: 18700,
    desc: "Regular buyers with consistent spend. Backbone of your revenue.",
    traits: ["Repeat purchasers", "Moderate AOV", "Category specific"],
    actions: ["Loyalty program tiers", "Cross-sell", "Personalized offers"],
  },
  {
    id: "potential",
    label: "Potential Loyalists",
    icon: "🌱",
    count: 964,
    pct: 25,
    avgSpend: 1180,
    frequency: 2.3,
    recency: 32,
    clv: 9200,
    desc: "Recent buyers with moderate spend. Strong conversion potential.",
    traits: ["Price sensitive", "Promo-driven", "Growing engagement"],
    actions: ["Onboarding series", "Targeted discounts", "Re-engagement emails"],
  },
  {
    id: "atrisk",
    label: "At Risk",
    icon: "⚠️",
    count: 523,
    pct: 14,
    avgSpend: 1890,
    frequency: 3.8,
    recency: 78,
    clv: 6100,
    desc: "Previously active but haven't purchased recently. Needs win-back.",
    traits: ["Lapsed buyers", "High historical value", "Responsive to offers"],
    actions: ["Win-back campaigns", "Special comeback offer", "Feedback survey"],
  },
  {
    id: "newcustomers",
    label: "New Customers",
    icon: "✨",
    count: 384,
    pct: 10,
    avgSpend: 620,
    frequency: 1.0,
    recency: 12,
    clv: 1400,
    desc: "Just acquired. Critical to convert to repeat buyers within 90 days.",
    traits: ["First-time buyers", "Exploring", "High churn risk"],
    actions: ["Welcome series", "First repeat incentive", "Product education"],
  },
];

const MONTHLY = [
  { m: "Jan", champions: 720, loyal: 1050, potential: 870, atrisk: 480, newcustomers: 310 },
  { m: "Feb", champions: 748, loyal: 1080, potential: 900, atrisk: 495, newcustomers: 330 },
  { m: "Mar", champions: 785, loyal: 1095, potential: 920, atrisk: 510, newcustomers: 350 },
  { m: "Apr", champions: 800, loyal: 1100, potential: 935, atrisk: 515, newcustomers: 360 },
  { m: "May", champions: 820, loyal: 1112, potential: 948, atrisk: 520, newcustomers: 372 },
  { m: "Jun", champions: 842, loyal: 1124, potential: 964, atrisk: 523, newcustomers: 384 },
];

const CATEGORIES = [
  { cat: "Electronics", champions: 38, loyal: 24, potential: 18, atrisk: 12, newcustomers: 8 },
  { cat: "Apparel", champions: 20, loyal: 32, potential: 25, atrisk: 15, newcustomers: 22 },
  { cat: "Home & Garden", champions: 28, loyal: 18, potential: 22, atrisk: 20, newcustomers: 14 },
  { cat: "Beauty", champions: 14, loyal: 26, potential: 35, atrisk: 18, newcustomers: 28 },
  { cat: "Sports", champions: 22, loyal: 20, potential: 18, atrisk: 24, newcustomers: 16 },
];

function ScatterPlot({ segments, activeSegment, setActiveSegment }) {
  const canvasRef = useRef(null);
  const [tooltip, setTooltip] = useState(null);

  const POINTS = {
    champions:    [{ x: 85, y: 88, r: 18 }, { x: 78, y: 82, r: 14 }, { x: 91, y: 79, r: 16 }],
    loyal:        [{ x: 65, y: 70, r: 16 }, { x: 72, y: 65, r: 12 }, { x: 60, y: 75, r: 14 }, { x: 68, y: 58, r: 11 }],
    potential:    [{ x: 45, y: 52, r: 13 }, { x: 55, y: 44, r: 11 }, { x: 40, y: 60, r: 12 }],
    atrisk:       [{ x: 30, y: 35, r: 12 }, { x: 22, y: 42, r: 10 }, { x: 35, y: 28, r: 11 }],
    newcustomers: [{ x: 70, y: 20, r: 9 }, { x: 62, y: 25, r: 8 }, { x: 78, y: 18, r: 10 }],
  };

  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext("2d");
    const W = canvas.width, H = canvas.height;
    const padL = 50, padB = 40, padT = 20, padR = 20;
    const plotW = W - padL - padR, plotH = H - padB - padT;

    ctx.clearRect(0, 0, W, H);

    // Grid
    ctx.strokeStyle = "rgba(136,135,128,0.15)";
    ctx.lineWidth = 1;
    for (let i = 0; i <= 4; i++) {
      const x = padL + (plotW * i) / 4;
      const y = padT + (plotH * i) / 4;
      ctx.beginPath(); ctx.moveTo(x, padT); ctx.lineTo(x, padT + plotH); ctx.stroke();
      ctx.beginPath(); ctx.moveTo(padL, y); ctx.lineTo(padL + plotW, y); ctx.stroke();
    }

    // Axis labels
    ctx.fillStyle = "#888780";
    ctx.font = "11px system-ui";
    ctx.textAlign = "center";
    ["0", "25", "50", "75", "100"].forEach((v, i) => {
      ctx.fillText(v, padL + (plotW * i) / 4, H - 10);
    });
    ctx.textAlign = "right";
    ["0", "25", "50", "75", "100"].forEach((v, i) => {
      ctx.fillText(v, padL - 6, padT + plotH - (plotH * i) / 4 + 4);
    });

    // Axis titles
    ctx.fillStyle = "#5F5E5A";
    ctx.font = "12px system-ui";
    ctx.textAlign = "center";
    ctx.fillText("Purchase Frequency →", padL + plotW / 2, H - 1);
    ctx.save();
    ctx.translate(14, padT + plotH / 2);
    ctx.rotate(-Math.PI / 2);
    ctx.fillText("Avg Spend →", 0, 0);
    ctx.restore();

    // Points
    Object.entries(POINTS).forEach(([seg, pts]) => {
      const color = COLORS[seg];
      const isActive = activeSegment === null || activeSegment === seg;
      pts.forEach(pt => {
        const cx = padL + (pt.x / 100) * plotW;
        const cy = padT + ((100 - pt.y) / 100) * plotH;
        ctx.beginPath();
        ctx.arc(cx, cy, pt.r, 0, Math.PI * 2);
        ctx.fillStyle = isActive
          ? color.bg + "cc"
          : color.bg + "33";
        ctx.fill();
        ctx.strokeStyle = isActive ? color.bg : color.bg + "55";
        ctx.lineWidth = 1.5;
        ctx.stroke();
      });
    });
  }, [activeSegment]);

  return (
    <div style={{ position: "relative" }}>
      <canvas ref={canvasRef} width={560} height={340} style={{ width: "100%", height: "auto" }} />
    </div>
  );
}

function RadarChart({ segment }) {
  const W = 220, H = 220, cx = 110, cy = 110, R = 80;
  const axes = ["Recency", "Frequency", "Spend", "Loyalty", "Growth"];
  const vals = {
    champions:    [92, 88, 95, 90, 72],
    loyal:        [74, 78, 70, 85, 65],
    potential:    [80, 45, 42, 55, 82],
    atrisk:       [25, 58, 62, 68, 30],
    newcustomers: [85, 18, 22, 20, 90],
  };
  const data = vals[segment] || vals.champions;
  const color = COLORS[segment];

  const toXY = (angle, r) => ({
    x: cx + r * Math.cos(angle - Math.PI / 2),
    y: cy + r * Math.sin(angle - Math.PI / 2),
  });

  const polyPoints = data.map((v, i) => {
    const angle = (i / axes.length) * 2 * Math.PI;
    const { x, y } = toXY(angle, (v / 100) * R);
    return `${x},${y}`;
  }).join(" ");

  return (
    <svg viewBox={`0 0 ${W} ${H}`} width={W} height={H}>
      {[0.25, 0.5, 0.75, 1].map(scale => (
        <polygon key={scale}
          points={axes.map((_, i) => {
            const angle = (i / axes.length) * 2 * Math.PI;
            const { x, y } = toXY(angle, R * scale);
            return `${x},${y}`;
          }).join(" ")}
          fill="none" stroke="rgba(136,135,128,0.2)" strokeWidth="1"
        />
      ))}
      {axes.map((_, i) => {
        const angle = (i / axes.length) * 2 * Math.PI;
        const { x, y } = toXY(angle, R);
        return <line key={i} x1={cx} y1={cy} x2={x} y2={y} stroke="rgba(136,135,128,0.2)" strokeWidth="1" />;
      })}
      <polygon points={polyPoints} fill={color.bg + "33"} stroke={color.bg} strokeWidth="2" />
      {axes.map((label, i) => {
        const angle = (i / axes.length) * 2 * Math.PI;
        const { x, y } = toXY(angle, R + 18);
        return (
          <text key={i} x={x} y={y} fontSize="10" textAnchor="middle" dominantBaseline="middle" fill="#888780">
            {label}
          </text>
        );
      })}
    </svg>
  );
}

function BarChart({ data, metric, colors }) {
  const max = Math.max(...data.map(d => d[metric]));
  return (
    <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
      {data.map(d => (
        <div key={d.cat} style={{ display: "flex", alignItems: "center", gap: 10 }}>
          <div style={{ width: 90, fontSize: 11, color: "var(--color-text-secondary)", textAlign: "right", flexShrink: 0 }}>
            {d.cat}
          </div>
          <div style={{ flex: 1, background: "var(--color-background-secondary)", borderRadius: 4, height: 20, overflow: "hidden" }}>
            <div style={{
              width: `${(d[metric] / max) * 100}%`,
              height: "100%",
              background: colors.bg,
              borderRadius: 4,
              transition: "width 0.5s ease",
            }} />
          </div>
          <div style={{ width: 28, fontSize: 11, color: "var(--color-text-secondary)" }}>{d[metric]}%</div>
        </div>
      ))}
    </div>
  );
}

function TrendMini({ data, segId }) {
  const vals = data.map(m => m[segId]);
  const min = Math.min(...vals), max = Math.max(...vals);
  const W = 80, H = 32;
  const pts = vals.map((v, i) => {
    const x = (i / (vals.length - 1)) * W;
    const y = H - ((v - min) / (max - min || 1)) * (H - 4) - 2;
    return `${x},${y}`;
  }).join(" ");
  const color = COLORS[segId];
  return (
    <svg width={W} height={H} viewBox={`0 0 ${W} ${H}`}>
      <polyline points={pts} fill="none" stroke={color.bg} strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" />
    </svg>
  );
}

export default function CustomerSegmentation() {
  const [activeSegment, setActiveSegment] = useState(null);
  const [view, setView] = useState("overview");
  const totalCustomers = SEGMENTS.reduce((s, seg) => s + seg.count, 0);

  const selectedSeg = activeSegment ? SEGMENTS.find(s => s.id === activeSegment) : null;

  return (
    <div style={{ fontFamily: "var(--font-sans)", color: "var(--color-text-primary)", padding: "1.5rem 0" }}>
      <h2 className="sr-only">Customer Segmentation Dashboard — RFM Analysis</h2>

      {/* Header */}
      <div style={{ display: "flex", alignItems: "baseline", justifyContent: "space-between", marginBottom: "1.5rem", flexWrap: "wrap", gap: 8 }}>
        <div>
          <div style={{ fontSize: 22, fontWeight: 500 }}>Customer Segmentation</div>
          <div style={{ fontSize: 13, color: "var(--color-text-secondary)", marginTop: 2 }}>
            RFM analysis · {totalCustomers.toLocaleString()} customers · June 2025
          </div>
        </div>
        <div style={{ display: "flex", gap: 6 }}>
          {["overview", "scatter", "categories"].map(v => (
            <button key={v} onClick={() => setView(v)} style={{
              padding: "6px 14px", fontSize: 13, borderRadius: "var(--border-radius-md)",
              border: view === v ? "1.5px solid var(--color-border-primary)" : "0.5px solid var(--color-border-tertiary)",
              background: view === v ? "var(--color-background-secondary)" : "transparent",
              cursor: "pointer", fontWeight: view === v ? 500 : 400,
              color: "var(--color-text-primary)",
            }}>
              {v === "overview" ? "Segments" : v === "scatter" ? "Scatter Plot" : "Categories"}
            </button>
          ))}
        </div>
      </div>

      {/* KPI row */}
      <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit, minmax(130px, 1fr))", gap: 10, marginBottom: "1.5rem" }}>
        {[
          { label: "Total customers", value: totalCustomers.toLocaleString(), sub: "+8.2% MoM" },
          { label: "Avg CLV", value: "$14,360", sub: "+12.4% YoY" },
          { label: "Champion share", value: "22%", sub: "842 customers" },
          { label: "At-risk rate", value: "14%", sub: "523 customers" },
        ].map(kpi => (
          <div key={kpi.label} style={{
            background: "var(--color-background-secondary)",
            borderRadius: "var(--border-radius-md)",
            padding: "0.75rem 1rem",
          }}>
            <div style={{ fontSize: 12, color: "var(--color-text-secondary)", marginBottom: 4 }}>{kpi.label}</div>
            <div style={{ fontSize: 22, fontWeight: 500 }}>{kpi.value}</div>
            <div style={{ fontSize: 11, color: "var(--color-text-secondary)", marginTop: 2 }}>{kpi.sub}</div>
          </div>
        ))}
      </div>

      {/* OVERVIEW TAB */}
      {view === "overview" && (
        <div>
          {/* Segment cards */}
          <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit, minmax(200px, 1fr))", gap: 10, marginBottom: "1.5rem" }}>
            {SEGMENTS.map(seg => {
              const color = COLORS[seg.id];
              const isActive = activeSegment === seg.id;
              return (
                <div key={seg.id} onClick={() => setActiveSegment(isActive ? null : seg.id)}
                  style={{
                    background: "var(--color-background-primary)",
                    border: isActive ? `2px solid ${color.bg}` : "0.5px solid var(--color-border-tertiary)",
                    borderRadius: "var(--border-radius-lg)",
                    padding: "1rem 1.25rem",
                    cursor: "pointer",
                    transition: "border 0.15s",
                  }}>
                  <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 8 }}>
                    <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
                      <span style={{ fontSize: 18 }}>{seg.icon}</span>
                      <span style={{ fontWeight: 500, fontSize: 14 }}>{seg.label}</span>
                    </div>
                    <TrendMini data={MONTHLY} segId={seg.id} />
                  </div>
                  <div style={{ display: "flex", alignItems: "baseline", gap: 6, marginBottom: 6 }}>
                    <span style={{ fontSize: 26, fontWeight: 500 }}>{seg.count.toLocaleString()}</span>
                    <span style={{ fontSize: 12, color: "var(--color-text-secondary)" }}>{seg.pct}% of base</span>
                  </div>
                  {/* Bar */}
                  <div style={{ background: "var(--color-background-secondary)", borderRadius: 3, height: 4, marginBottom: 10 }}>
                    <div style={{ width: `${seg.pct * 3.3}%`, height: "100%", background: color.bg, borderRadius: 3 }} />
                  </div>
                  <div style={{ display: "flex", justifyContent: "space-between", fontSize: 11, color: "var(--color-text-secondary)" }}>
                    <span>Avg spend: <strong style={{ color: "var(--color-text-primary)" }}>${seg.avgSpend.toLocaleString()}</strong></span>
                    <span>CLV: <strong style={{ color: "var(--color-text-primary)" }}>${(seg.clv / 1000).toFixed(1)}k</strong></span>
                  </div>
                </div>
              );
            })}
          </div>

          {/* Detail panel */}
          {selectedSeg && (() => {
            const color = COLORS[selectedSeg.id];
            return (
              <div style={{
                background: "var(--color-background-primary)",
                border: `0.5px solid ${color.border}`,
                borderLeft: `4px solid ${color.bg}`,
                borderRadius: "var(--border-radius-lg)",
                padding: "1.25rem",
                display: "grid",
                gridTemplateColumns: "1fr 200px",
                gap: "1.5rem",
              }}>
                <div>
                  <div style={{ display: "flex", alignItems: "center", gap: 10, marginBottom: 10 }}>
                    <span style={{ fontSize: 24 }}>{selectedSeg.icon}</span>
                    <div>
                      <div style={{ fontWeight: 500, fontSize: 16 }}>{selectedSeg.label}</div>
                      <div style={{ fontSize: 12, color: "var(--color-text-secondary)" }}>{selectedSeg.desc}</div>
                    </div>
                  </div>
                  <div style={{ display: "grid", gridTemplateColumns: "repeat(4, 1fr)", gap: 10, marginBottom: 14 }}>
                    {[
                      { label: "Customers", val: selectedSeg.count.toLocaleString() },
                      { label: "Avg spend", val: `$${selectedSeg.avgSpend.toLocaleString()}` },
                      { label: "Frequency", val: `${selectedSeg.frequency}x/yr` },
                      { label: "Recency", val: `${selectedSeg.recency}d ago` },
                    ].map(m => (
                      <div key={m.label} style={{
                        background: color.light, borderRadius: "var(--border-radius-md)",
                        padding: "8px 12px",
                      }}>
                        <div style={{ fontSize: 11, color: color.text, opacity: 0.7 }}>{m.label}</div>
                        <div style={{ fontSize: 15, fontWeight: 500, color: color.text }}>{m.val}</div>
                      </div>
                    ))}
                  </div>
                  <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 16 }}>
                    <div>
                      <div style={{ fontSize: 11, fontWeight: 500, color: "var(--color-text-secondary)", marginBottom: 6, textTransform: "uppercase", letterSpacing: "0.05em" }}>Traits</div>
                      {selectedSeg.traits.map(t => (
                        <div key={t} style={{ display: "flex", alignItems: "center", gap: 8, marginBottom: 5, fontSize: 13 }}>
                          <span style={{ width: 6, height: 6, borderRadius: "50%", background: color.bg, flexShrink: 0 }} />
                          {t}
                        </div>
                      ))}
                    </div>
                    <div>
                      <div style={{ fontSize: 11, fontWeight: 500, color: "var(--color-text-secondary)", marginBottom: 6, textTransform: "uppercase", letterSpacing: "0.05em" }}>Recommended actions</div>
                      {selectedSeg.actions.map(a => (
                        <div key={a} style={{ display: "flex", alignItems: "center", gap: 8, marginBottom: 5, fontSize: 13 }}>
                          <span style={{ fontSize: 12, color: color.bg }}>→</span>
                          {a}
                        </div>
                      ))}
                    </div>
                  </div>
                </div>
                <div style={{ display: "flex", flexDirection: "column", alignItems: "center" }}>
                  <div style={{ fontSize: 11, fontWeight: 500, color: "var(--color-text-secondary)", marginBottom: 4, textTransform: "uppercase", letterSpacing: "0.05em" }}>RFM profile</div>
                  <RadarChart segment={selectedSeg.id} />
                </div>
              </div>
            );
          })()}

          {/* Donut summary */}
          <div style={{ marginTop: "1.5rem", background: "var(--color-background-primary)", border: "0.5px solid var(--color-border-tertiary)", borderRadius: "var(--border-radius-lg)", padding: "1.25rem" }}>
            <div style={{ fontSize: 14, fontWeight: 500, marginBottom: 12 }}>Segment distribution</div>
            <div style={{ display: "flex", alignItems: "center", gap: 24 }}>
              <svg viewBox="0 0 140 140" width={140} height={140} style={{ flexShrink: 0 }}>
                {(() => {
                  let cum = 0;
                  const R = 50, r = 30, cx = 70, cy = 70;
                  return SEGMENTS.map(seg => {
                    const frac = seg.pct / 100;
                    const start = cum * 2 * Math.PI - Math.PI / 2;
                    const end = (cum + frac) * 2 * Math.PI - Math.PI / 2;
                    const x1 = cx + R * Math.cos(start), y1 = cy + R * Math.sin(start);
                    const x2 = cx + R * Math.cos(end), y2 = cy + R * Math.sin(end);
                    const ix1 = cx + r * Math.cos(start), iy1 = cy + r * Math.sin(start);
                    const ix2 = cx + r * Math.cos(end), iy2 = cy + r * Math.sin(end);
                    const large = frac > 0.5 ? 1 : 0;
                    const path = `M ${x1} ${y1} A ${R} ${R} 0 ${large} 1 ${x2} ${y2} L ${ix2} ${iy2} A ${r} ${r} 0 ${large} 0 ${ix1} ${iy1} Z`;
                    cum += frac;
                    return (
                      <path key={seg.id} d={path} fill={COLORS[seg.id].bg}
                        opacity={activeSegment === null || activeSegment === seg.id ? 1 : 0.25}
                        onClick={() => setActiveSegment(activeSegment === seg.id ? null : seg.id)}
                        style={{ cursor: "pointer", transition: "opacity 0.2s" }}
                      />
                    );
                  });
                })()}
              </svg>
              <div style={{ flex: 1 }}>
                {SEGMENTS.map(seg => (
                  <div key={seg.id} style={{ display: "flex", alignItems: "center", gap: 8, marginBottom: 8, cursor: "pointer" }}
                    onClick={() => setActiveSegment(activeSegment === seg.id ? null : seg.id)}>
                    <span style={{ width: 10, height: 10, borderRadius: 2, background: COLORS[seg.id].bg, flexShrink: 0 }} />
                    <span style={{ fontSize: 13, flex: 1 }}>{seg.icon} {seg.label}</span>
                    <span style={{ fontSize: 12, color: "var(--color-text-secondary)" }}>{seg.count.toLocaleString()}</span>
                    <span style={{ fontSize: 11, fontWeight: 500, background: COLORS[seg.id].light, color: COLORS[seg.id].text, padding: "2px 8px", borderRadius: "var(--border-radius-md)" }}>{seg.pct}%</span>
                  </div>
                ))}
              </div>
            </div>
          </div>
        </div>
      )}

      {/* SCATTER TAB */}
      {view === "scatter" && (
        <div style={{ background: "var(--color-background-primary)", border: "0.5px solid var(--color-border-tertiary)", borderRadius: "var(--border-radius-lg)", padding: "1.25rem" }}>
          <div style={{ fontSize: 14, fontWeight: 500, marginBottom: 4 }}>Frequency vs. spend cluster map</div>
          <div style={{ fontSize: 12, color: "var(--color-text-secondary)", marginBottom: 12 }}>Bubble size = customer lifetime value · Click segment legend to filter</div>
          <div style={{ display: "flex", flexWrap: "wrap", gap: 10, marginBottom: 14 }}>
            {SEGMENTS.map(seg => (
              <button key={seg.id} onClick={() => setActiveSegment(activeSegment === seg.id ? null : seg.id)}
                style={{
                  display: "flex", alignItems: "center", gap: 6,
                  padding: "4px 12px", borderRadius: "var(--border-radius-md)",
                  border: activeSegment === seg.id ? `1.5px solid ${COLORS[seg.id].bg}` : "0.5px solid var(--color-border-tertiary)",
                  background: activeSegment === seg.id ? COLORS[seg.id].light : "transparent",
                  cursor: "pointer", fontSize: 12,
                  color: activeSegment === seg.id ? COLORS[seg.id].text : "var(--color-text-primary)",
                }}>
                <span style={{ width: 8, height: 8, borderRadius: "50%", background: COLORS[seg.id].bg }} />
                {seg.label}
              </button>
            ))}
          </div>
          <ScatterPlot segments={SEGMENTS} activeSegment={activeSegment} setActiveSegment={setActiveSegment} />
        </div>
      )}

      {/* CATEGORIES TAB */}
      {view === "categories" && (
        <div>
          <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit, minmax(240px, 1fr))", gap: 12 }}>
            {SEGMENTS.map(seg => (
              <div key={seg.id} style={{
                background: "var(--color-background-primary)",
                border: "0.5px solid var(--color-border-tertiary)",
                borderRadius: "var(--border-radius-lg)",
                padding: "1rem 1.25rem",
              }}>
                <div style={{ display: "flex", alignItems: "center", gap: 8, marginBottom: 12 }}>
                  <span style={{ width: 10, height: 10, borderRadius: 2, background: COLORS[seg.id].bg }} />
                  <span style={{ fontWeight: 500, fontSize: 14 }}>{seg.icon} {seg.label}</span>
                </div>
                <BarChart data={CATEGORIES} metric={seg.id} colors={COLORS[seg.id]} />
              </div>
            ))}
          </div>
          <div style={{ marginTop: 12, fontSize: 12, color: "var(--color-text-secondary)", padding: "8px 12px", background: "var(--color-background-secondary)", borderRadius: "var(--border-radius-md)" }}>
            Values show % of each segment spending in that category. Champions concentrate in Electronics; New Customers skew Beauty & Apparel.
          </div>
        </div>
      )}
    </div>
  );
}
