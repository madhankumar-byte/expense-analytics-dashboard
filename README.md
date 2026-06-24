import { useState, useMemo } from "react";
import {
  PieChart, Pie, Cell, Tooltip, ResponsiveContainer,
  BarChart, Bar, XAxis, YAxis, CartesianGrid,
} from "recharts";

// ─── Constants ───────────────────────────────────────────────────────────────

const CATEGORIES = [
  "Food", "Transport", "Shopping", "Health",
  "Entertainment", "Utilities", "Rent", "Other",
];

const CATEGORY_COLORS = {
  Food: "#f97316",
  Transport: "#3b82f6",
  Shopping: "#a855f7",
  Health: "#22c55e",
  Entertainment: "#ec4899",
  Utilities: "#eab308",
  Rent: "#06b6d4",
  Other: "#94a3b8",
};

const CATEGORY_ICONS = {
  Food: "🍔", Transport: "🚗", Shopping: "🛍️", Health: "💊",
  Entertainment: "🎬", Utilities: "💡", Rent: "🏠", Other: "📦",
};

const INITIAL_TRANSACTIONS = [
  { id: 1,  type: "income",  title: "Monthly Salary",       amount: 50000, category: "Other",         date: "2026-06-01" },
  { id: 2,  type: "expense", title: "Rent Payment",          amount: 15000, category: "Rent",          date: "2026-06-02" },
  { id: 3,  type: "expense", title: "Grocery Shopping",      amount: 3200,  category: "Food",          date: "2026-06-05" },
  { id: 4,  type: "expense", title: "Metro Pass",            amount: 500,   category: "Transport",     date: "2026-06-06" },
  { id: 5,  type: "expense", title: "Netflix Subscription",  amount: 649,   category: "Entertainment", date: "2026-06-10" },
  { id: 6,  type: "expense", title: "Doctor Visit",          amount: 800,   category: "Health",        date: "2026-06-12" },
  { id: 7,  type: "income",  title: "Freelance Project",     amount: 12000, category: "Other",         date: "2026-06-15" },
  { id: 8,  type: "expense", title: "Electricity Bill",      amount: 1200,  category: "Utilities",     date: "2026-06-18" },
  { id: 9,  type: "expense", title: "Online Shopping",       amount: 2500,  category: "Shopping",      date: "2026-06-20" },
  { id: 10, type: "expense", title: "Dinner Out",            amount: 1800,  category: "Food",          date: "2026-06-22" },
];

const EMPTY_FORM = {
  type: "expense",
  title: "",
  amount: "",
  category: "Food",
  date: new Date().toISOString().split("T")[0],
};

// ─── Helpers ─────────────────────────────────────────────────────────────────

const fmt = (n) => "₹" + Number(n).toLocaleString("en-IN");

const tooltipStyle = {
  background: "#0f172a",
  border: "1px solid #334155",
  borderRadius: 8,
  color: "#e2e8f0",
  fontSize: 13,
};

// ─── Main Component ───────────────────────────────────────────────────────────

export default function App() {
  const [transactions, setTransactions] = useState(INITIAL_TRANSACTIONS);
  const [form, setForm]                 = useState(EMPTY_FORM);
  const [filterType, setFilterType]     = useState("all");
  const [filterCat, setFilterCat]       = useState("all");
  const [activeTab, setActiveTab]       = useState("overview");
  const [editId, setEditId]             = useState(null);
  const [errors, setErrors]             = useState({});

  // ── Derived values ──────────────────────────────────────────────────────────
  const totalIncome = useMemo(
    () => transactions.filter((t) => t.type === "income").reduce((s, t) => s + t.amount, 0),
    [transactions]
  );
  const totalExpense = useMemo(
    () => transactions.filter((t) => t.type === "expense").reduce((s, t) => s + t.amount, 0),
    [transactions]
  );
  const balance = totalIncome - totalExpense;

  const categoryData = useMemo(() => {
    const map = {};
    transactions
      .filter((t) => t.type === "expense")
      .forEach((t) => { map[t.category] = (map[t.category] || 0) + t.amount; });
    return Object.entries(map)
      .map(([name, value]) => ({ name, value }))
      .sort((a, b) => b.value - a.value);
  }, [transactions]);

  const filteredTransactions = useMemo(() => {
    return transactions
      .filter((t) => filterType === "all" || t.type === filterType)
      .filter((t) => filterCat  === "all" || t.category === filterCat)
      .sort((a, b) => new Date(b.date) - new Date(a.date));
  }, [transactions, filterType, filterCat]);

  // ── Handlers ────────────────────────────────────────────────────────────────
  const validate = () => {
    const e = {};
    if (!form.title.trim())                              e.title  = "Title is required";
    if (!form.amount || isNaN(form.amount) || Number(form.amount) <= 0) e.amount = "Enter a valid amount";
    if (!form.date)                                      e.date   = "Date is required";
    setErrors(e);
    return Object.keys(e).length === 0;
  };

  const handleSubmit = () => {
    if (!validate()) return;
    const entry = { ...form, amount: Number(form.amount) };
    if (editId !== null) {
      setTransactions((prev) => prev.map((t) => (t.id === editId ? { ...entry, id: editId } : t)));
      setEditId(null);
    } else {
      setTransactions((prev) => [...prev, { ...entry, id: Date.now() }]);
    }
    setForm(EMPTY_FORM);
    setErrors({});
  };

  const handleEdit = (t) => {
    setEditId(t.id);
    setForm({ type: t.type, title: t.title, amount: String(t.amount), category: t.category, date: t.date });
    setActiveTab("add");
  };

  const handleDelete  = (id) => setTransactions((prev) => prev.filter((t) => t.id !== id));
  const handleCancel  = () => { setEditId(null); setForm(EMPTY_FORM); setErrors({}); };
  const setField      = (key) => (e) => setForm((f) => ({ ...f, [key]: e.target.value }));
  const setDirectField = (key, val) => setForm((f) => ({ ...f, [key]: val }));

  // ── Styles (shared) ─────────────────────────────────────────────────────────
  const card = (bg, accent) => ({
    background: bg,
    border: `1px solid ${accent}33`,
    borderRadius: 14,
    padding: "20px 24px",
  });

  const inputStyle = (hasErr) => ({
    width: "100%",
    padding: "11px 14px",
    background: "#0f172a",
    border: `1px solid ${hasErr ? "#ef4444" : "#334155"}`,
    borderRadius: 10,
    color: "#e2e8f0",
    fontSize: 14,
    outline: "none",
    boxSizing: "border-box",
    colorScheme: "dark",
  });

  const errMsg = (key) =>
    errors[key] ? (
      <div style={{ color: "#ef4444", fontSize: 12, marginTop: 4 }}>{errors[key]}</div>
    ) : null;

  // ── Render ───────────────────────────────────────────────────────────────────
  return (
    <div style={{ background: "#0f172a", minHeight: "100vh", color: "#e2e8f0", paddingBottom: 48 }}>

      {/* ── Header ── */}
      <div style={{ background: "linear-gradient(135deg,#1e293b 0%,#0f172a 100%)", borderBottom: "1px solid #1e293b", padding: "24px 24px 0" }}>
        <div style={{ maxWidth: 1100, margin: "0 auto" }}>
          <div style={{ display: "flex", alignItems: "center", gap: 12, marginBottom: 4 }}>
            <span style={{ fontSize: 30 }}>💰</span>
            <div>
              <h1 style={{ margin: 0, fontSize: 22, fontWeight: 700, color: "#f8fafc", letterSpacing: "-0.5px" }}>
                Expense Analytics
              </h1>
              <p style={{ margin: 0, fontSize: 13, color: "#64748b" }}>
                Track income, expenses &amp; spending patterns
              </p>
            </div>
          </div>

          {/* Tabs */}
          <div style={{ display: "flex", gap: 4, marginTop: 20 }}>
            {[
              ["overview",      "📊 Overview"],
              ["transactions",  "📋 Transactions"],
              ["add",           editId ? "✏️ Edit" : "➕ Add New"],
            ].map(([key, label]) => (
              <button
                key={key}
                onClick={() => setActiveTab(key)}
                style={{
                  padding: "10px 18px",
                  border: "none",
                  borderRadius: "8px 8px 0 0",
                  cursor: "pointer",
                  background: activeTab === key ? "#1e40af" : "transparent",
                  color:      activeTab === key ? "#fff"    : "#64748b",
                  fontWeight: activeTab === key ? 600       : 400,
                  fontSize: 14,
                }}
              >
                {label}
              </button>
            ))}
          </div>
        </div>
      </div>

      <div style={{ maxWidth: 1100, margin: "0 auto", padding: "24px 24px 0" }}>

        {/* ── Summary Cards (always visible) ── */}
        <div style={{ display: "grid", gridTemplateColumns: "repeat(3,1fr)", gap: 16, marginBottom: 28 }}>
          {[
            { label: "Total Income",   value: totalIncome,  color: "#22c55e", bg: "#052e16", icon: "↑" },
            { label: "Total Expenses", value: totalExpense, color: "#ef4444", bg: "#2d0a0a", icon: "↓" },
            { label: "Net Balance",    value: balance,      color: balance >= 0 ? "#38bdf8" : "#fb923c", bg: balance >= 0 ? "#082032" : "#2d1500", icon: "=" },
          ].map((c) => (
            <div key={c.label} style={card(c.bg, c.color)}>
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 8 }}>
                <span style={{ fontSize: 12, color: "#64748b", textTransform: "uppercase", letterSpacing: "0.08em", fontWeight: 600 }}>
                  {c.label}
                </span>
                <span style={{ fontSize: 18, background: c.color + "22", borderRadius: 6, padding: "2px 8px", color: c.color }}>
                  {c.icon}
                </span>
              </div>
              <div style={{ fontSize: 26, fontWeight: 700, color: c.color }}>{fmt(Math.abs(c.value))}</div>
              <div style={{ fontSize: 12, color: "#475569", marginTop: 4 }}>
                {transactions.filter((t) =>
                  c.label === "Total Income"   ? t.type === "income"  :
                  c.label === "Total Expenses" ? t.type === "expense" : true
                ).length} transactions
              </div>
            </div>
          ))}
        </div>

        {/* ═══════════════════════════════ OVERVIEW TAB ═══════════════════════ */}
        {activeTab === "overview" && (
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 24 }}>

            {/* Pie Chart */}
            <div style={{ background: "#1e293b", borderRadius: 14, padding: 24, border: "1px solid #334155" }}>
              <h3 style={{ margin: "0 0 20px", fontSize: 15, fontWeight: 600, color: "#cbd5e1" }}>
                Spending by Category
              </h3>
              {categoryData.length > 0 ? (
                <ResponsiveContainer width="100%" height={260}>
                  <PieChart>
                    <Pie
                      data={categoryData} cx="50%" cy="50%" outerRadius={90}
                      dataKey="value"
                      label={({ name, percent }) => `${name} ${(percent * 100).toFixed(0)}%`}
                      labelLine={false} fontSize={11}
                    >
                      {categoryData.map((entry) => (
                        <Cell key={entry.name} fill={CATEGORY_COLORS[entry.name] || "#94a3b8"} />
                      ))}
                    </Pie>
                    <Tooltip formatter={(v) => fmt(v)} contentStyle={tooltipStyle} />
                  </PieChart>
                </ResponsiveContainer>
              ) : (
                <p style={{ color: "#475569", textAlign: "center", padding: 40 }}>No expense data yet.</p>
              )}
            </div>

            {/* Bar Chart */}
            <div style={{ background: "#1e293b", borderRadius: 14, padding: 24, border: "1px solid #334155" }}>
              <h3 style={{ margin: "0 0 20px", fontSize: 15, fontWeight: 600, color: "#cbd5e1" }}>
                Category Breakdown
              </h3>
              {categoryData.length > 0 ? (
                <ResponsiveContainer width="100%" height={260}>
                  <BarChart data={categoryData} margin={{ top: 5, right: 10, left: 10, bottom: 30 }}>
                    <CartesianGrid strokeDasharray="3 3" stroke="#334155" />
                    <XAxis dataKey="name" tick={{ fill: "#64748b", fontSize: 11 }} angle={-30} textAnchor="end" />
                    <YAxis tick={{ fill: "#64748b", fontSize: 11 }} tickFormatter={(v) => "₹" + (v / 1000).toFixed(0) + "k"} />
                    <Tooltip formatter={(v) => fmt(v)} contentStyle={tooltipStyle} />
                    <Bar dataKey="value" radius={[6, 6, 0, 0]}>
                      {categoryData.map((entry) => (
                        <Cell key={entry.name} fill={CATEGORY_COLORS[entry.name] || "#94a3b8"} />
                      ))}
                    </Bar>
                  </BarChart>
                </ResponsiveContainer>
              ) : (
                <p style={{ color: "#475569", textAlign: "center", padding: 40 }}>No data yet.</p>
              )}
            </div>

            {/* Category Summary Grid */}
            <div style={{ gridColumn: "1 / -1", background: "#1e293b", borderRadius: 14, padding: 24, border: "1px solid #334155" }}>
              <h3 style={{ margin: "0 0 16px", fontSize: 15, fontWeight: 600, color: "#cbd5e1" }}>
                Category Summary
              </h3>
              {categoryData.length === 0 ? (
                <p style={{ color: "#475569", textAlign: "center", padding: 20 }}>No expenses recorded yet.</p>
              ) : (
                <div style={{ display: "grid", gridTemplateColumns: "repeat(4,1fr)", gap: 12 }}>
                  {categoryData.map((cat) => (
                    <div
                      key={cat.name}
                      style={{ background: "#0f172a", borderRadius: 10, padding: "14px 16px", border: `1px solid ${CATEGORY_COLORS[cat.name]}33` }}
                    >
                      <div style={{ display: "flex", alignItems: "center", gap: 8, marginBottom: 6 }}>
                        <span style={{ fontSize: 18 }}>{CATEGORY_ICONS[cat.name]}</span>
                        <span style={{ fontSize: 13, fontWeight: 600, color: "#94a3b8" }}>{cat.name}</span>
                      </div>
                      <div style={{ fontSize: 18, fontWeight: 700, color: CATEGORY_COLORS[cat.name] }}>
                        {fmt(cat.value)}
                      </div>
                      <div style={{ fontSize: 11, color: "#475569", marginTop: 2 }}>
                        {totalExpense > 0 ? ((cat.value / totalExpense) * 100).toFixed(1) : 0}% of expenses
                      </div>
                    </div>
                  ))}
                </div>
              )}
            </div>
          </div>
        )}

        {/* ═══════════════════════════ TRANSACTIONS TAB ═══════════════════════ */}
        {activeTab === "transactions" && (
          <div>
            {/* Filters */}
            <div style={{ display: "flex", gap: 12, marginBottom: 20, flexWrap: "wrap", alignItems: "center" }}>
              <div style={{ display: "flex", gap: 4, background: "#1e293b", padding: 4, borderRadius: 10, border: "1px solid #334155" }}>
                {[["all","All"],["income","Income"],["expense","Expense"]].map(([val, label]) => (
                  <button
                    key={val}
                    onClick={() => setFilterType(val)}
                    style={{
                      padding: "6px 14px", border: "none", borderRadius: 7, cursor: "pointer", fontSize: 13, fontWeight: 500,
                      background: filterType === val ? "#1e40af" : "transparent",
                      color:      filterType === val ? "#fff"    : "#64748b",
                    }}
                  >
                    {label}
                  </button>
                ))}
              </div>

              <select
                value={filterCat}
                onChange={(e) => setFilterCat(e.target.value)}
                style={{ padding: "8px 14px", background: "#1e293b", border: "1px solid #334155", borderRadius: 10, color: "#e2e8f0", fontSize: 13, cursor: "pointer" }}
              >
                <option value="all">All Categories</option>
                {CATEGORIES.map((c) => (
                  <option key={c} value={c}>{CATEGORY_ICONS[c]} {c}</option>
                ))}
              </select>

              <div style={{ marginLeft: "auto", fontSize: 13, color: "#64748b" }}>
                {filteredTransactions.length} record{filteredTransactions.length !== 1 ? "s" : ""}
              </div>
            </div>

            {/* List */}
            <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
              {filteredTransactions.length === 0 ? (
                <div style={{ textAlign: "center", padding: 60, color: "#475569", background: "#1e293b", borderRadius: 14, border: "1px solid #334155" }}>
                  No transactions found. Add one using the "Add New" tab.
                </div>
              ) : (
                filteredTransactions.map((t) => (
                  <div
                    key={t.id}
                    style={{ display: "flex", alignItems: "center", gap: 16, background: "#1e293b", borderRadius: 12, padding: "14px 20px", border: "1px solid #334155" }}
                  >
                    <div style={{ width: 42, height: 42, borderRadius: 10, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 20, background: (CATEGORY_COLORS[t.category] || "#94a3b8") + "22", flexShrink: 0 }}>
                      {CATEGORY_ICONS[t.category] || "📦"}
                    </div>
                    <div style={{ flex: 1 }}>
                      <div style={{ fontWeight: 600, fontSize: 14, color: "#f1f5f9" }}>{t.title}</div>
                      <div style={{ fontSize: 12, color: "#475569", marginTop: 3 }}>
                        <span style={{ background: (CATEGORY_COLORS[t.category] || "#94a3b8") + "22", color: CATEGORY_COLORS[t.category] || "#94a3b8", padding: "2px 8px", borderRadius: 20, fontWeight: 500, fontSize: 11 }}>
                          {t.category}
                        </span>
                        <span style={{ marginLeft: 8 }}>
                          {new Date(t.date).toLocaleDateString("en-IN", { day: "numeric", month: "short", year: "numeric" })}
                        </span>
                      </div>
                    </div>
                    <div style={{ fontSize: 17, fontWeight: 700, color: t.type === "income" ? "#22c55e" : "#ef4444", minWidth: 110, textAlign: "right" }}>
                      {t.type === "income" ? "+" : "-"}{fmt(t.amount)}
                    </div>
                    <div style={{ display: "flex", gap: 6, flexShrink: 0 }}>
                      <button
                        onClick={() => handleEdit(t)}
                        style={{ padding: "6px 12px", background: "#1e3a5f", border: "1px solid #1e40af", borderRadius: 7, color: "#60a5fa", fontSize: 12, cursor: "pointer", fontWeight: 500 }}
                      >
                        Edit
                      </button>
                      <button
                        onClick={() => handleDelete(t.id)}
                        style={{ padding: "6px 12px", background: "#2d0a0a", border: "1px solid #7f1d1d", borderRadius: 7, color: "#f87171", fontSize: 12, cursor: "pointer", fontWeight: 500 }}
                      >
                        Delete
                      </button>
                    </div>
                  </div>
                ))
              )}
            </div>
          </div>
        )}

        {/* ═══════════════════════════ ADD / EDIT TAB ═════════════════════════ */}
        {activeTab === "add" && (
          <div style={{ maxWidth: 560, margin: "0 auto" }}>
            <div style={{ background: "#1e293b", borderRadius: 16, padding: 32, border: "1px solid #334155" }}>
              <h2 style={{ margin: "0 0 24px", fontSize: 18, fontWeight: 700, color: "#f1f5f9" }}>
                {editId ? "✏️ Edit Transaction" : "➕ Add Transaction"}
              </h2>

              {/* Type Toggle */}
              <div style={{ marginBottom: 20 }}>
                <label style={{ fontSize: 12, fontWeight: 600, color: "#64748b", textTransform: "uppercase", letterSpacing: "0.08em", display: "block", marginBottom: 8 }}>
                  Type
                </label>
                <div style={{ display: "flex", gap: 10 }}>
                  {["income", "expense"].map((type) => (
                    <button
                      key={type}
                      onClick={() => setDirectField("type", type)}
                      style={{
                        flex: 1, padding: 12, border: "2px solid", cursor: "pointer", borderRadius: 10, fontSize: 14, fontWeight: 600,
                        borderColor: form.type === type ? (type === "income" ? "#22c55e" : "#ef4444") : "#334155",
                        background:  form.type === type ? (type === "income" ? "#052e16" : "#2d0a0a") : "transparent",
                        color:       form.type === type ? (type === "income" ? "#22c55e" : "#ef4444") : "#64748b",
                      }}
                    >
                      {type === "income" ? "↑ Income" : "↓ Expense"}
                    </button>
                  ))}
                </div>
              </div>

              {/* Title */}
              <div style={{ marginBottom: 16 }}>
                <label style={{ fontSize: 12, fontWeight: 600, color: "#64748b", textTransform: "uppercase", letterSpacing: "0.08em", display: "block", marginBottom: 6 }}>
                  Title
                </label>
                <input
                  value={form.title}
                  onChange={setField("title")}
                  placeholder="e.g. Monthly Salary, Grocery..."
                  style={inputStyle(errors.title)}
                />
                {errMsg("title")}
              </div>

              {/* Amount */}
              <div style={{ marginBottom: 16 }}>
                <label style={{ fontSize: 12, fontWeight: 600, color: "#64748b", textTransform: "uppercase", letterSpacing: "0.08em", display: "block", marginBottom: 6 }}>
                  Amount (₹)
                </label>
                <input
                  type="number"
                  value={form.amount}
                  onChange={setField("amount")}
                  placeholder="0"
                  style={inputStyle(errors.amount)}
                />
                {errMsg("amount")}
              </div>

              {/* Category */}
              <div style={{ marginBottom: 16 }}>
                <label style={{ fontSize: 12, fontWeight: 600, color: "#64748b", textTransform: "uppercase", letterSpacing: "0.08em", display: "block", marginBottom: 8 }}>
                  Category
                </label>
                <div style={{ display: "grid", gridTemplateColumns: "repeat(4,1fr)", gap: 8 }}>
                  {CATEGORIES.map((cat) => (
                    <button
                      key={cat}
                      onClick={() => setDirectField("category", cat)}
                      style={{
                        padding: "8px 4px", border: "2px solid", cursor: "pointer", borderRadius: 9, fontSize: 11, fontWeight: 500,
                        borderColor: form.category === cat ? CATEGORY_COLORS[cat] : "#334155",
                        background:  form.category === cat ? CATEGORY_COLORS[cat] + "22" : "transparent",
                        color:       form.category === cat ? CATEGORY_COLORS[cat] : "#64748b",
                      }}
                    >
                      {CATEGORY_ICONS[cat]} {cat}
                    </button>
                  ))}
                </div>
              </div>

              {/* Date */}
              <div style={{ marginBottom: 28 }}>
                <label style={{ fontSize: 12, fontWeight: 600, color: "#64748b", textTransform: "uppercase", letterSpacing: "0.08em", display: "block", marginBottom: 6 }}>
                  Date
                </label>
                <input
                  type="date"
                  value={form.date}
                  onChange={setField("date")}
                  style={inputStyle(errors.date)}
                />
                {errMsg("date")}
              </div>

              {/* Submit / Cancel */}
              <div style={{ display: "flex", gap: 10 }}>
                <button
                  onClick={handleSubmit}
                  style={{
                    flex: 1, padding: 13,
                    background: form.type === "income" ? "#166534" : "#1e40af",
                    border: "none", borderRadius: 10, color: "#fff", fontSize: 15, fontWeight: 700, cursor: "pointer",
                  }}
                >
                  {editId ? "Save Changes" : `Add ${form.type === "income" ? "Income" : "Expense"}`}
                </button>
                {editId && (
                  <button
                    onClick={handleCancel}
                    style={{ padding: "13px 20px", background: "#1e293b", border: "1px solid #334155", borderRadius: 10, color: "#94a3b8", fontSize: 14, cursor: "pointer" }}
                  >
                    Cancel
                  </button>
                )}
              </div>
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
