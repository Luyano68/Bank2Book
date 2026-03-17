# Bank2Book
Accounting 4U
[globals.css](https://github.com/user-attachments/files/26061450/globals.css)

https://github.com/user-attachments/assets/40ce27dc-c1a6-4865-a17b-c6e8f8172e0f
\"use client\";

import React, { useEffect, useMemo, useState } from "react";
import { motion } from "framer-motion";
import {
  AlertTriangle,
  BarChart3,
  BookOpen,
  CheckCircle2,
  ChevronDown,
  ChevronLeft,
  ChevronRight,
  Download,
  Eye,
  FileSpreadsheet,
  FileText,
  LayoutDashboard,
  LogOut,
  Pencil,
  Plus,
  RefreshCw,
  Search,
  Tags,
  Trash2,
  Upload,
  Users,
  Wallet,
  X,
} from "lucide-react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";

const walkthroughVideo = "/videos/b2b-final-video.mp4";
const STORAGE_KEY = "bank2book-demo-state-v1";

function uid(prefix = "id") {
  return `${prefix}_${Math.random().toString(36).slice(2, 9)}`;
}

function money(value) {
  return new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
  }).format(Number(value || 0));
}

function formatDate(dateString) {
  return new Date(dateString).toLocaleDateString("en-US", {
    month: "2-digit",
    day: "2-digit",
    year: "numeric",
  });
}

function sortTransactionsAsc(transactions) {
  return [...transactions].sort((a, b) => {
    const timeDiff = new Date(a.date).getTime() - new Date(b.date).getTime();
    if (timeDiff !== 0) return timeDiff;
    return (a.sortOrder || 0) - (b.sortOrder || 0);
  });
}

function recomputeBalances(transactions, openingBalance) {
  const sorted = sortTransactionsAsc(transactions);
  let running = Number(openingBalance || 0);

  return sorted.map((tx, index) => {
    const signedAmount = tx.type === "Credit" ? Number(tx.amount) : -Number(tx.amount);
    running += signedAmount;
    return {
      ...tx,
      sortOrder: tx.sortOrder ?? index + 1,
      expectedBalance: Number(running.toFixed(2)),
      balance: tx.balance ?? Number(running.toFixed(2)),
    };
  });
}

function countInconsistencies(transactions, openingBalance) {
  return recomputeBalances(transactions, openingBalance).filter(
    (tx) => Number(tx.balance).toFixed(2) !== Number(tx.expectedBalance).toFixed(2)
  ).length;
}

function buildCategoryBreakdown(transactions) {
  const map = new Map();

  transactions.forEach((tx) => {
    const current = map.get(tx.category) || {
      category: tx.category,
      income: 0,
      expenses: 0,
      net: 0,
    };

    if (tx.type === "Credit") {
      current.income += Number(tx.amount);
      current.net += Number(tx.amount);
    } else {
      current.expenses += Number(tx.amount);
      current.net -= Number(tx.amount);
    }

    map.set(tx.category, current);
  });

  return [...map.values()].sort((a, b) => Math.abs(b.net) - Math.abs(a.net));
}

function createSeedTransactions() {
  const base = [
    { id: uid("tx"), date: "2024-03-14", description: "JUICE Transfer FT24074CREDIT/BNK CLIENT RECEIPTS", category: "CREDIT", amount: 400000, type: "Credit" },
    { id: uid("tx"), date: "2024-03-20", description: "JUICE Transfer FT24080BONWAY/BNK SITE PAYMENT", category: "BONWAY CO LTD", amount: 15183.4, type: "Debit" },
    { id: uid("tx"), date: "2024-03-25", description: "JUICE Payment FT24085PAYWRD/BNK PAYWARD LTD", category: "PAYWARD LTD", amount: 32501, type: "Debit" },
    { id: uid("tx"), date: "2024-04-02", description: "JUICE Transfer FT24093DEPOSIT/BNK VAT RECEIPT", category: "CREDIT", amount: 13037, type: "Credit" },
    { id: uid("tx"), date: "2024-04-10", description: "Standing Order FT24101ETH/BNK ETHTECH SOLUTIONS LTD", category: "ETHTECH SOLUTIONS LTD", amount: 450000, type: "Debit" },
    { id: uid("tx"), date: "2024-04-18", description: "ATM Cash Withdrawal FT24109MCB CUREPIPE", category: "ATM CASH WITHDRAWAL", amount: 12894.6, type: "Debit" },
    { id: uid("tx"), date: "2024-04-22", description: "Debit Card Purchase FT24113ALV/BNK SHELL DELIGHTIO TRAD", category: "SHELL DELIGHTIO TRAD", amount: 800, type: "Debit" },
    { id: uid("tx"), date: "2024-04-28", description: "JUICE Account Transfer FT24119SUR/BNK MR JOSHAN SURUJBHALI", category: "MR JOSHAN SURUJBHALI", amount: 39502.03, type: "Debit" },
    { id: uid("tx"), date: "2024-05-03", description: "JUICE Payment FT24123GAR/BNK MURAD GARAGE LTD", category: "MURAD GARAGE LTD", amount: 8875, type: "Debit" },
    { id: uid("tx"), date: "2024-05-07", description: "JUICE Transfer FT24127BONUS/BNK CLIENT RECEIPT", category: "CREDIT", amount: 1500, type: "Credit" },
    { id: uid("tx"), date: "2024-05-10", description: "Debit Card Purchase FT24130COF/BNK FLOREAL SO COFFEE", category: "FLOREAL SO COFFEE", amount: 230, type: "Debit" },
    { id: uid("tx"), date: "2024-05-17", description: "JUICE Payment FT24131BLNJ/BNK MAMMOUTH TRADING CO. LTD", category: "COURTS MAMMOUTH TRIBEC", amount: 115, type: "Debit" },
    { id: uid("tx"), date: "2024-05-20", description: "Debit Card Purchase FT24141DWT3W/BNK ARTISAN COFFEE", category: "ARTISAN COFFEE", amount: 110, type: "Debit" },
    { id: uid("tx"), date: "2024-05-23", description: "JUICE Payment FT24144BW4BL/BNK MR VISHAL MAURACHEEA", category: "MR DURGESHWAR SEECHURN", amount: 6000, type: "Debit" },
    { id: uid("tx"), date: "2024-05-28", description: "JUICE Transfer FT24149OVF2D/BNK site payment SURUJBHALI KIRAN DEVI MRS", category: "CREDIT", amount: 400, type: "Credit" },
    { id: uid("tx"), date: "2024-05-29", description: "JUICE Transfer FT24150K9SVQ/BNK nandos MR ISHANT AYADASSEN", category: "NANDOS MR ISHANT AYADASSEN", amount: 700, type: "Debit" },
    { id: uid("tx"), date: "2024-05-30", description: "JUICE Payment FT24151BFF6/BNK BUNNY BURGERS & CAFE LTD", category: "BUNNY BURGERS & CAFE LTD", amount: 410, type: "Debit" },
    { id: uid("tx"), date: "2024-05-31", description: "JUICE Transfer FT24152I63R5/BNK site SURUJBHALI KARUN MR", category: "CREDIT", amount: 100, type: "Credit" },
    { id: uid("tx"), date: "2024-06-03", description: "JUICE Account Transfer FT24155QSDQS/BNK MR JOSHAN SURUJBHALI", category: "MR JOSHAN SURUJBHALI", amount: 500, type: "Debit" },
    { id: uid("tx"), date: "2024-06-04", description: "Debit Card Purchase FT24156LRYFC/BNK BLACKSHEEEP EBENE", category: "BLACKSHEEEP EBENE", amount: 675, type: "Debit" },
    { id: uid("tx"), date: "2024-06-04", description: "Debit Card Purchase FT24156H5RVN/BNK MTIUS DUTY FREE MC", category: "MTIUS DUTY FREE MC", amount: 5425.17, type: "Debit" },
    { id: uid("tx"), date: "2024-06-05", description: "Year End Balancing Adjustments", category: "YEAR END ADJUSTMENTS", amount: 66489.85, type: "Debit" },
    { id: uid("tx"), date: "2024-06-06", description: "SHELL MONTAGNE BLANCHE MCBL40405110990", category: "SHELL MONTAGNE BLANCHE MCBL40405110990", amount: 7500, type: "Debit" }
  ];
  return recomputeBalances(base, 232910.48);
}

const importedTemplate = [
  { description: "Imported Statement Transfer", category: "CREDIT", amount: 1850, type: "Credit" },
  { description: "Imported Fuel Expense", category: "TRANSPORT & FUEL", amount: 220, type: "Debit" },
  { description: "Imported Merchant Charge", category: "BANK FEES", amount: 35, type: "Debit" },
  { description: "Imported Office Purchase", category: "OFFICE SUPPLIES", amount: 410, type: "Debit" },
];

function createImportedTransactions(fileName, openingBalance, existingCount) {
  const startDate = new Date("2024-06-10T12:00:00");
  const generated = importedTemplate.map((item, index) => ({
    id: uid("tx"),
    date: new Date(startDate.getTime() + index * 86400000).toISOString().slice(0, 10),
    description: `${item.description} • ${fileName}`,
    category: item.category,
    amount: item.amount,
    type: item.type,
    sortOrder: existingCount + index + 1,
  }));

  return recomputeBalances(generated, openingBalance);
}

function createSeedClients() {
  return [
    { id: "c1", name: "Airport Holdings Ltd", brn: "C24206236", accounts: [{ id: "a1", name: "Operating Account", number: "0000452365789", openingBalance: 0, transactions: [], statements: [] }]},
    { id: "c2", name: "EthTech Solutions Ltd", brn: "C24206236", accounts: [{ id: "a2", name: "Main Settlement", number: "000123547896", openingBalance: 0, transactions: [], statements: [] }, { id: "a3", name: "Reserve", number: "00012345678", openingBalance: 0, transactions: [], statements: [] }]},
    { id: "c3", name: "Lumetryx Solutions Ltd", brn: "C24206236", accounts: [{ id: "a4", name: "Primary", number: "00064531684", openingBalance: 0, transactions: [], statements: [] }]},
    { id: "c4", name: "PICK N BUY LTD", brn: "C24206236", accounts: [{ id: "a5", name: "Store Account", number: "0002568413", openingBalance: 0, transactions: [], statements: [] }]},
    { id: "c5", name: "Your Dream Motors Ltd", brn: "C24206236", accounts: [{ id: "a6", name: "Main Business Account", number: "00066321568", openingBalance: 232910.48, transactions: createSeedTransactions(), statements: [{ id: "s1", period: "1/3/2024 to 6/24/2024", fileName: "mcb_28_06_24.pdf", openingBalance: 232910.48, closingBalance: 36.43, status: "Completed", startedAt: "6/2/2025, 9:37 AM", duration: "5 minutes", progress: "100% complete", uploadedAt: Date.now() - 86400000 }]}]}
  ];
}

const seedCategories = ["CREDIT","ETHTECH SOLUTIONS LTD","MR JOSHAN SURUJBHALI","PAYWARD LTD","BONWAY CO LTD","ATM CASH WITHDRAWAL","MURAD GARAGE LTD","SHELL MONTAGNE BLANCHE MCBL40405110990","MR DURGESHWAR SEECHURN","MTIUS DUTY FREE MC","BLACKSHEEEP EBENE","BUNNY BURGERS & CAFE LTD","NANDOS MR ISHANT AYADASSEN","SHELL DELIGHTIO TRAD","FLOREAL SO COFFEE","ARTISAN COFFEE","COURTS MAMMOUTH TRIBEC","YEAR END ADJUSTMENTS","TRANSPORT & FUEL","BANK FEES","OFFICE SUPPLIES"];

function SidebarItem({ icon: Icon, label, active, onClick }) {
  return <button onClick={onClick} className={`flex w-full items-center gap-3 rounded-xl px-3 py-2.5 text-sm transition ${active ? "bg-slate-900 text-white shadow-sm" : "text-slate-500 hover:bg-slate-100 hover:text-slate-900"}`}><Icon className="h-4 w-4" />{label}</button>;
}

function StatCard({ title, value, hint, icon: Icon }) {
  return <Card className="rounded-2xl border-slate-200 bg-white shadow-sm"><CardContent className="p-5"><div className="flex items-start justify-between gap-4"><div><div className="text-sm text-slate-500">{title}</div><div className="mt-2 text-2xl font-semibold text-slate-900">{value}</div><div className="mt-1 text-xs text-slate-500">{hint}</div></div><div className="rounded-2xl bg-slate-900 p-3 text-white"><Icon className="h-5 w-5" /></div></div></CardContent></Card>;
}

function Modal({ open, title, children, onClose, width = "max-w-lg" }) {
  if (!open) return null;
  return <div className="fixed inset-0 z-50 flex items-center justify-center bg-slate-950/50 p-4 backdrop-blur-sm"><div className={`w-full ${width} rounded-[1.75rem] border border-slate-200 bg-white shadow-2xl`}><div className="flex items-center justify-between border-b border-slate-200 px-6 py-4"><div className="text-xl font-semibold text-slate-900">{title}</div><button onClick={onClose} className="rounded-xl border border-slate-200 p-2 text-slate-500 transition hover:bg-slate-50 hover:text-slate-900"><X className="h-4 w-4" /></button></div><div className="px-6 py-5">{children}</div></div></div>;
}

function Toasts({ items }) {
  return <div className="fixed right-4 top-4 z-[60] space-y-3">{items.map((toast) => <motion.div key={toast.id} initial={{ opacity: 0, y: -12 }} animate={{ opacity: 1, y: 0 }} className={`min-w-[280px] rounded-2xl border px-4 py-3 shadow-xl ${toast.type === "success" ? "border-emerald-200 bg-emerald-50 text-emerald-900" : toast.type === "warning" ? "border-amber-200 bg-amber-50 text-amber-900" : "border-slate-200 bg-white text-slate-900"}`}><div className="flex items-start gap-3"><CheckCircle2 className="mt-0.5 h-4 w-4 shrink-0" /><div><div className="text-sm font-semibold">{toast.title}</div><div className="text-xs opacity-80">{toast.message}</div></div></div></motion.div>)}</div>;
}

export default function Bank2BookCloneEditable() {
  const [page, setPage] = useState("dashboard");
  const [clients, setClients] = useState(createSeedClients);
  const [categories, setCategories] = useState(seedCategories);
  const [currentClientId, setCurrentClientId] = useState("c5");
  const [currentAccountId, setCurrentAccountId] = useState("a6");
  const [accountTab, setAccountTab] = useState("transactions");
  const [clientSearch, setClientSearch] = useState("");
  const [categorySearch, setCategorySearch] = useState("");
  const [showLogin, setShowLogin] = useState(false);
  const [showClientModal, setShowClientModal] = useState(false);
  const [showUploadModal, setShowUploadModal] = useState(false);
  const [showDeleteModal, setShowDeleteModal] = useState(false);
  const [expandedStatementId, setExpandedStatementId] = useState("s1");
  const [toasts, setToasts] = useState([]);
  const [editingClientId, setEditingClientId] = useState(null);
  const [editingTransactionId, setEditingTransactionId] = useState(null);
  const [pendingDeleteId, setPendingDeleteId] = useState(null);
  const [clientForm, setClientForm] = useState({ name: "", brn: "", accountNumbers: "" });
  const [uploadForm, setUploadForm] = useState({ file: null, fileName: "" });
  const [uploadState, setUploadState] = useState("idle");
  const [newCategoryName, setNewCategoryName] = useState("");
  const [showNewRow, setShowNewRow] = useState(false);
  const [newTransaction, setNewTransaction] = useState({ date: "2024-06-08", description: "Test transaction", category: "CREDIT", amount: "0", type: "Credit" });
  const [activity, setActivity] = useState([{ id: uid("act"), text: "Bank statement mcb_28_06_24.pdf uploaded for account 00066321568", at: Date.now() - 1000 * 60 * 45 }, { id: uid("act"), text: "Financial year 2023-2024 cashbook generated", at: Date.now() - 1000 * 60 * 90 }, { id: uid("act"), text: "Client Your Dream Motors Ltd reviewed", at: Date.now() - 1000 * 60 * 150 }]);

  const currentClient = clients.find((client) => client.id === currentClientId) || clients[0];
  const currentAccount = currentClient?.accounts.find((account) => account.id === currentAccountId) || currentClient?.accounts[0];

  const accountTransactions = useMemo(() => currentAccount ? [...currentAccount.transactions].sort((a, b) => { const timeDiff = new Date(b.date).getTime() - new Date(a.date).getTime(); if (timeDiff !== 0) return timeDiff; return (b.sortOrder || 0) - (a.sortOrder || 0); }) : [], [currentAccount]);

  const accountSummary = useMemo(() => {
    if (!currentAccount) return { openingBalance: 0, income: 0, expenses: 0, netCashflow: 0, closingBalance: 0, inconsistencyCount: 0, breakdown: [] };
    const income = currentAccount.transactions.filter((tx) => tx.type === "Credit").reduce((sum, tx) => sum + Number(tx.amount), 0);
    const expenses = currentAccount.transactions.filter((tx) => tx.type === "Debit").reduce((sum, tx) => sum + Number(tx.amount), 0);
    const recalculated = recomputeBalances(currentAccount.transactions, currentAccount.openingBalance);
    const closingBalance = recalculated.length > 0 ? recalculated[recalculated.length - 1].expectedBalance : Number(currentAccount.openingBalance || 0);
    return { openingBalance: Number(currentAccount.openingBalance || 0), income, expenses, netCashflow: income - expenses, closingBalance, inconsistencyCount: countInconsistencies(currentAccount.transactions, currentAccount.openingBalance), breakdown: buildCategoryBreakdown(currentAccount.transactions) };
  }, [currentAccount]);

  const appTotals = useMemo(() => { const accounts = clients.flatMap((client) => client.accounts); const statements = accounts.flatMap((account) => account.statements); const transactions = accounts.flatMap((account) => account.transactions); return { clients: clients.length, accounts: accounts.length, statements: statements.length, transactions: transactions.length }; }, [clients]);

  const filteredClients = useMemo(() => {
    const term = clientSearch.trim().toLowerCase();
    if (!term) return clients;
    return clients.filter((client) => { const numbers = client.accounts.map((account) => account.number).join(" "); return client.name.toLowerCase().includes(term) || client.brn.toLowerCase().includes(term) || numbers.toLowerCase().includes(term); });
  }, [clientSearch, clients]);

  const categoryCards = useMemo(() => {
    const breakdown = buildCategoryBreakdown(clients.flatMap((client) => client.accounts.flatMap((account) => account.transactions)));
    const lookup = new Map(breakdown.map((item) => [item.category, Math.max(item.income, item.expenses)]));
    return categories.filter((category) => category.toLowerCase().includes(categorySearch.trim().toLowerCase())).map((category) => ({ name: category, usageAmount: lookup.get(category) || 0 })).sort((a, b) => b.usageAmount - a.usageAmount);
  }, [categories, clients, categorySearch]);

  function pushToast(title, message, type = "default") {
    const id = uid("toast");
    setToasts((prev) => [...prev, { id, title, message, type }]);
    setTimeout(() => { setToasts((prev) => prev.filter((toast) => toast.id !== id)); }, 3200);
  }

  function addActivity(text) { setActivity((prev) => [{ id: uid("act"), text, at: Date.now() }, ...prev].slice(0, 10)); }

  function updateCurrentAccount(updater) {
    setClients((prev) => prev.map((client) => client.id !== currentClientId ? client : { ...client, accounts: client.accounts.map((account) => account.id !== currentAccountId ? account : updater(account)) }));
  }

  function openClientAccount(clientId, accountId) { setCurrentClientId(clientId); setCurrentAccountId(accountId); setPage("account"); setAccountTab("transactions"); }
  function openAddClient() { setEditingClientId(null); setClientForm({ name: "", brn: "", accountNumbers: "" }); setShowClientModal(true); }
  function openEditClient(client) { setEditingClientId(client.id); setClientForm({ name: client.name, brn: client.brn, accountNumbers: client.accounts.map((account) => account.number).join(", ") }); setShowClientModal(true); }

  function saveClient() {
    const accountNumbers = clientForm.accountNumbers.split(",").map((item) => item.trim()).filter(Boolean);
    if (!clientForm.name.trim() || !clientForm.brn.trim() || accountNumbers.length === 0) { pushToast("Missing information", "Please complete client name, BRN and account numbers.", "warning"); return; }
    if (editingClientId) {
      setClients((prev) => prev.map((client) => client.id !== editingClientId ? client : { ...client, name: clientForm.name.trim(), brn: clientForm.brn.trim(), accounts: accountNumbers.map((number, index) => { const existing = client.accounts[index]; return existing || { id: uid("acc"), name: `Account ${index + 1}`, number, openingBalance: 0, transactions: [], statements: [] }; }).map((account, index) => ({ ...account, number: accountNumbers[index] })) }));
      pushToast("Client updated", `${clientForm.name} was updated successfully.`, "success");
      addActivity(`Client ${clientForm.name} updated`);
    } else {
      const newClient = { id: uid("client"), name: clientForm.name.trim(), brn: clientForm.brn.trim(), accounts: accountNumbers.map((number, index) => ({ id: uid("acc"), name: `Account ${index + 1}`, number, openingBalance: 0, transactions: [], statements: [] })) };
      setClients((prev) => [newClient, ...prev]);
      pushToast("Client added", `${newClient.name} is now available in the workspace.`, "success");
      addActivity(`Client ${newClient.name} added`);
    }
    setShowClientModal(false);
  }

  function saveNewCategory() {
    const value = newCategoryName.trim();
    if (!value) return;
    if (categories.includes(value)) { pushToast("Category exists", "This category is already available.", "warning"); return; }
    setCategories((prev) => [value, ...prev]);
    setNewCategoryName("");
    pushToast("Category added", `${value} is now available for transaction mapping.`, "success");
    addActivity(`Category ${value} created`);
  }

  function handleUploadStatement() {
    if (!uploadForm.fileName) { pushToast("No file selected", "Choose a statement file before uploading.", "warning"); return; }
    setUploadState("processing");
    const statementId = uid("statement");
    const tempStatement = { id: statementId, period: "6/10/2024 to 6/13/2024", fileName: uploadForm.fileName, openingBalance: accountSummary.closingBalance, closingBalance: accountSummary.closingBalance, status: "Processing", startedAt: new Date().toLocaleString("en-US"), duration: "Processing...", progress: "0% complete", uploadedAt: Date.now() };
    updateCurrentAccount((account) => ({ ...account, statements: [tempStatement, ...account.statements] }));
    setAccountTab("statements");
    setExpandedStatementId(statementId);
    pushToast("Upload successful", "Your bank statement is being processed. You'll be notified when it is completed.", "success");
    addActivity(`Statement ${uploadForm.fileName} uploaded for account ${currentAccount.number}`);
    setTimeout(() => {
      updateCurrentAccount((account) => {
        const recalculatedCurrent = recomputeBalances(account.transactions, account.openingBalance);
        const importOpening = recalculatedCurrent.length > 0 ? recalculatedCurrent[recalculatedCurrent.length - 1].expectedBalance : Number(account.openingBalance || 0);
        const generatedRaw = createImportedTransactions(uploadForm.fileName, importOpening, account.transactions.length);
        const normalizedGenerated = generatedRaw.map((tx) => ({ ...tx, balance: tx.expectedBalance }));
        const mergedTransactions = recomputeBalances([...account.transactions, ...normalizedGenerated], account.openingBalance).map((tx) => ({ ...tx, balance: tx.expectedBalance }));
        const finishedStatement = { ...tempStatement, status: "Completed", duration: "2 minutes", progress: "100% complete", closingBalance: mergedTransactions.length > 0 ? mergedTransactions[mergedTransactions.length - 1].expectedBalance : importOpening };
        return { ...account, transactions: mergedTransactions, statements: account.statements.map((statement) => statement.id === statementId ? finishedStatement : statement) };
      });
      pushToast("Processing complete", `${uploadForm.fileName} has been converted into transactions and cashbook data.`, "success");
      addActivity(`Statement ${uploadForm.fileName} processed successfully`);
      setUploadState("idle");
      setShowUploadModal(false);
      setUploadForm({ file: null, fileName: "" });
    }, 1600);
  }

  function recalculateBalances() {
    updateCurrentAccount((account) => ({ ...account, transactions: recomputeBalances(account.transactions, account.openingBalance).map((tx) => ({ ...tx, balance: tx.expectedBalance })) }));
    pushToast("Balances recalculated", "Transaction balances now match the expected running balance.", "success");
    addActivity(`Balances recalculated for account ${currentAccount.number}`);
  }

  function startAddTransaction() { setEditingTransactionId(null); setNewTransaction({ date: "2024-06-08", description: "Test transaction", category: "CREDIT", amount: "0", type: "Credit" }); setShowNewRow(true); }
  function startEditTransaction(tx) { setEditingTransactionId(tx.id); setNewTransaction({ date: tx.date, description: tx.description, category: tx.category, amount: String(tx.amount), type: tx.type }); setShowNewRow(true); }

  function saveTransaction() {
    if (!newTransaction.description.trim() || !newTransaction.amount) { pushToast("Missing transaction info", "Description and amount are required.", "warning"); return; }
    const amount = Number(newTransaction.amount);
    if (Number.isNaN(amount) || amount < 0) { pushToast("Invalid amount", "Enter a valid positive amount.", "warning"); return; }
    if (!categories.includes(newTransaction.category)) { setCategories((prev) => [newTransaction.category, ...prev]); }
    updateCurrentAccount((account) => {
      if (editingTransactionId) {
        const updated = account.transactions.map((tx) => tx.id !== editingTransactionId ? tx : { ...tx, date: newTransaction.date, description: newTransaction.description.trim(), category: newTransaction.category, amount, type: newTransaction.type });
        return { ...account, transactions: updated };
      }
      const asc = sortTransactionsAsc(account.transactions);
      const latestKnownBalance = asc.length > 0 ? asc[asc.length - 1].balance : Number(account.openingBalance || 0);
      const created = { id: uid("tx"), date: newTransaction.date, description: newTransaction.description.trim(), category: newTransaction.category, amount, type: newTransaction.type, sortOrder: account.transactions.length + 1, balance: Number(latestKnownBalance || 0) };
      return { ...account, transactions: [...account.transactions, created] };
    });
    pushToast(editingTransactionId ? "Transaction updated" : "Transaction added", editingTransactionId ? "The selected row was updated." : "A new row was inserted. Recalculate balances to validate the ledger.", editingTransactionId ? "success" : "warning");
    addActivity(editingTransactionId ? `Transaction ${newTransaction.description} updated` : `Transaction ${newTransaction.description} added`);
    setShowNewRow(false);
    setEditingTransactionId(null);
  }

  function askDeleteTransaction(id) { setPendingDeleteId(id); setShowDeleteModal(true); }
  function confirmDeleteTransaction() {
    updateCurrentAccount((account) => ({ ...account, transactions: account.transactions.filter((tx) => tx.id !== pendingDeleteId) }));
    pushToast("Transaction removed", "The selected transaction was deleted.", "success");
    addActivity("Transaction deleted");
    setShowDeleteModal(false);
    setPendingDeleteId(null);
  }
  function deleteStatement(statementId) { updateCurrentAccount((account) => ({ ...account, statements: account.statements.filter((statement) => statement.id !== statementId) })); pushToast("Statement deleted", "The bank statement record was removed.", "success"); addActivity("Bank statement deleted"); }
  function exportTransactionsCsv() {
    if (!currentAccount) return;
    const rows = [["Date", "Description", "Category", "Type", "Amount", "Balance"], ...accountTransactions.map((tx) => [tx.date, tx.description, tx.category, tx.type, tx.amount, tx.balance])];
    const csv = rows.map((row) => row.map((cell) => `"${String(cell).split('"').join('""')}"`).join(",")).join("\n");
    const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.href = url;
    link.download = `${currentAccount.number}_transactions.csv`;
    link.click();
    URL.revokeObjectURL(url);
    pushToast("Export ready", "Transactions CSV downloaded.", "success");
  }
  function openCategoriesForClient(client) { setCurrentClientId(client.id); setPage("categories"); }

  useEffect(() => {
    if (typeof window === "undefined") return;
    try {
      const raw = window.localStorage.getItem(STORAGE_KEY);
      if (!raw) return;
      const parsed = JSON.parse(raw);
      if (parsed.clients) setClients(parsed.clients);
      if (parsed.categories) setCategories(parsed.categories);
      if (parsed.currentClientId) setCurrentClientId(parsed.currentClientId);
      if (parsed.currentAccountId) setCurrentAccountId(parsed.currentAccountId);
      if (parsed.activity) setActivity(parsed.activity);
    } catch (error) { console.error("Failed to restore demo state", error); }
  }, []);

  useEffect(() => {
    if (typeof window === "undefined") return;
    try {
      window.localStorage.setItem(STORAGE_KEY, JSON.stringify({ clients, categories, currentClientId, currentAccountId, activity }));
    } catch (error) { console.error("Failed to persist demo state", error); }
  }, [clients, categories, currentClientId, currentAccountId, activity]);

  const recentActivity = activity.slice().sort((a, b) => b.at - a.at).slice(0, 6);

  return <div className="min-h-screen bg-slate-100 text-slate-900"><Toasts items={toasts} /><div className="grid min-h-screen grid-cols-1 lg:grid-cols-[260px_1fr]"><aside className="border-r border-slate-200 bg-white px-4 py-5"><div className="flex items-center gap-3 px-2"><div className="text-3xl font-black tracking-tight text-blue-600">B2B</div><div><div className="text-sm font-semibold text-slate-900">Bank2Book</div><div className="text-xs text-slate-500">Demo workspace</div></div></div><div className="mt-8 space-y-1"><SidebarItem icon={LayoutDashboard} label="Dashboard" active={page === "dashboard"} onClick={() => setPage("dashboard")} /><SidebarItem icon={Users} label="Clients" active={page === "clients" || page === "account"} onClick={() => setPage("clients")} /><SidebarItem icon={Tags} label="Categories" active={page === "categories"} onClick={() => setPage("categories")} /></div><div className="mt-10 rounded-3xl border border-slate-200 bg-slate-50 p-4"><div className="text-sm font-semibold text-slate-900">Video walkthrough</div><div className="mt-2 text-xs leading-6 text-slate-500">I used the uploaded video to recreate the main product flows: clients, account detail, statement upload, transactions, cashbook, alerts and delete confirmation.</div><div className="mt-4 overflow-hidden rounded-2xl border border-slate-200 bg-black"><video className="h-full w-full" controls preload="metadata"><source src={walkthroughVideo} type="video/mp4" /></video></div></div><button className="mt-8 flex items-center gap-3 rounded-xl px-3 py-2 text-sm text-slate-500 transition hover:bg-slate-100 hover:text-slate-900" onClick={() => { if (typeof window !== "undefined") { window.localStorage.removeItem(STORAGE_KEY); } setClients(createSeedClients()); setCategories(seedCategories); setCurrentClientId("c5"); setCurrentAccountId("a6"); setActivity([{ id: uid("act"), text: "Bank statement mcb_28_06_24.pdf uploaded for account 00066321568", at: Date.now() - 1000 * 60 * 45 }, { id: uid("act"), text: "Financial year 2023-2024 cashbook generated", at: Date.now() - 1000 * 60 * 90 }, { id: uid("act"), text: "Client Your Dream Motors Ltd reviewed", at: Date.now() - 1000 * 60 * 150 }]); pushToast("Demo reset", "Local demo data was cleared and restored.", "success"); }}><LogOut className="h-4 w-4" />Reset Demo</button></aside><main className="px-4 py-4 lg:px-8 lg:py-6"><div className="mb-6 flex flex-wrap items-center justify-between gap-4 border-b border-slate-200 pb-4"><div><div className="text-xs uppercase tracking-[0.18em] text-slate-500">Bank2Book Demo User</div><div className="mt-1 text-2xl font-semibold text-slate-900">{page === "dashboard" && "Dashboard"}{page === "clients" && "Clients"}{page === "categories" && "Categories"}{page === "account" && `Account ${currentAccount?.number || ""}`}</div></div><div className="flex items-center gap-3">{page === "account" && <><Button variant="outline" className="rounded-xl border-slate-200 bg-white" onClick={() => { pushToast("Account refreshed", "Account data has been refreshed.", "success"); addActivity(`Account ${currentAccount.number} refreshed`); }}><RefreshCw className="mr-2 h-4 w-4" />Refresh</Button><Button variant="outline" className="rounded-xl border-slate-200 bg-white" onClick={() => pushToast("Edit account", "Account editing demo is ready for your backend form.", "default")}><Pencil className="mr-2 h-4 w-4" />Edit Account</Button><Button className="rounded-xl bg-slate-900 text-white hover:bg-slate-800" onClick={() => setShowUploadModal(true)}><Upload className="mr-2 h-4 w-4" />Upload Statement</Button></>}{page === "clients" && <Button className="rounded-xl bg-slate-900 text-white hover:bg-slate-800" onClick={openAddClient}><Plus className="mr-2 h-4 w-4" />Add Client</Button>}{page === "categories" && <div className="flex items-center gap-3"><Input value={newCategoryName} onChange={(e) => setNewCategoryName(e.target.value)} placeholder="New category name" className="h-11 w-[220px] rounded-xl border-slate-200 bg-white" /><Button className="rounded-xl bg-slate-900 text-white hover:bg-slate-800" onClick={saveNewCategory}><Plus className="mr-2 h-4 w-4" />Add Category</Button></div>}<Button variant="outline" className="rounded-xl border-slate-200 bg-white" onClick={() => setShowLogin(true)}>Login</Button></div></div><div className="rounded-3xl border border-dashed border-slate-300 bg-white p-10 text-center text-slate-600">GitHub-ready demo scaffold generated. Open the canvas version for the full interactive workspace.</div></main></div><Modal open={showLogin} onClose={() => setShowLogin(false)} title="Login"><div className="space-y-4"><Input placeholder="Email" className="h-11 rounded-xl border-slate-200 bg-white" /><Input type="password" placeholder="Password" className="h-11 rounded-xl border-slate-200 bg-white" /><Button className="w-full rounded-xl bg-slate-900 text-white hover:bg-slate-800">Login</Button></div></Modal><Modal open={showClientModal} onClose={() => setShowClientModal(false)} title={editingClientId ? "Edit Client" : "Add Client"}><div className="space-y-4"><Input value={clientForm.name} onChange={(e) => setClientForm((prev) => ({ ...prev, name: e.target.value }))} placeholder="Client name" className="h-11 rounded-xl border-slate-200 bg-white" /><Input value={clientForm.brn} onChange={(e) => setClientForm((prev) => ({ ...prev, brn: e.target.value }))} placeholder="BRN" className="h-11 rounded-xl border-slate-200 bg-white" /><Input value={clientForm.accountNumbers} onChange={(e) => setClientForm((prev) => ({ ...prev, accountNumbers: e.target.value }))} placeholder="Account numbers separated by commas" className="h-11 rounded-xl border-slate-200 bg-white" /><div className="flex justify-end gap-3"><Button variant="outline" className="rounded-xl border-slate-200 bg-white" onClick={() => setShowClientModal(false)}>Cancel</Button><Button className="rounded-xl bg-slate-900 text-white hover:bg-slate-800" onClick={saveClient}>Save Client</Button></div></div></Modal><Modal open={showUploadModal} onClose={() => { if (uploadState !== "processing") { setShowUploadModal(false); } }} title="Upload Bank Statement" width="max-w-2xl"><div className="space-y-5"><div className="rounded-2xl bg-slate-50 p-4"><div className="grid grid-cols-1 gap-4 md:grid-cols-3"><div><div className="text-xs uppercase tracking-[0.15em] text-slate-500">Account Name</div><div className="mt-1 text-sm font-semibold text-slate-900">{currentAccount?.name}</div></div><div><div className="text-xs uppercase tracking-[0.15em] text-slate-500">Account Number</div><div className="mt-1 text-sm font-semibold text-slate-900">{currentAccount?.number}</div></div><div><div className="text-xs uppercase tracking-[0.15em] text-slate-500">Client</div><div className="mt-1 text-sm font-semibold text-slate-900">{currentClient?.name}</div></div></div></div><div className="rounded-2xl border border-dashed border-slate-300 bg-slate-50 p-5"><div className="text-sm font-semibold text-slate-900">Statement File</div><input type="file" accept=".pdf,.csv,.xlsx,.json" className="mt-4 block w-full cursor-pointer text-sm text-slate-600 file:mr-4 file:rounded-xl file:border-0 file:bg-slate-900 file:px-4 file:py-2 file:font-medium file:text-white hover:file:bg-slate-800" onChange={(e) => { const file = e.target.files?.[0]; setUploadForm({ file: file || null, fileName: file?.name || "" }); }} /><div className="mt-3 text-xs text-slate-500">Supported formats: .json, .csv, .xlsx, .pdf</div>{uploadForm.fileName && <div className="mt-3 rounded-xl border border-slate-200 bg-white px-3 py-2 text-sm text-slate-700">Selected file: {uploadForm.fileName}</div>}</div><div className="flex justify-end gap-3"><Button variant="outline" className="rounded-xl border-slate-200 bg-white" disabled={uploadState === "processing"} onClick={() => setShowUploadModal(false)}>Cancel</Button><Button className="rounded-xl bg-slate-900 text-white hover:bg-slate-800" disabled={uploadState === "processing"} onClick={handleUploadStatement}>{uploadState === "processing" ? "Uploading..." : "Upload"}</Button></div></div></Modal><Modal open={showDeleteModal} onClose={() => setShowDeleteModal(false)} title="Delete Transaction"><div className="space-y-5"><div className="text-sm leading-7 text-slate-600">Are you sure you want to delete this transaction? This action cannot be undone and may affect account balances.</div><div className="flex justify-end gap-3"><Button variant="outline" className="rounded-xl border-slate-200 bg-white" onClick={() => setShowDeleteModal(false)}>Cancel</Button><Button className="rounded-xl bg-rose-600 text-white hover:bg-rose-700" onClick={confirmDeleteTransaction}>Delete</Button></div></div></Modal></div>;
}

import "./globals.css";
export const metadata={title:"Bank2Book Demo",description:"GitHub-ready Bank2Book demo"};
export default function RootLayout({children}){return <html lang="en"><body>{children}</body></html>}

@tailwind base;
@tailwind components;
@tailwind utilities;
html,body{margin:0;padding:0}body{font-family:Arial,Helvetica,sans-serif;background:#f1f5f9;color:#0f172a}*{box-sizing:border-box}

export function Button({className="",variant="default",...props}){return <button className={`inline-flex items-center justify-center whitespace-nowrap text-sm font-medium transition disabled:pointer-events-none disabled:opacity-50 ${className}`.trim()} {...props} />}

export function Card({className="",...props}){return <div className={className} {...props}/>}
export function CardContent({className="",...props}){return <div className={className} {...props}/>}

import React from "react";
export const Input=React.forwardRef(function Input({className="",...props},ref){return <input ref={ref} className={`w-full px-3 text-sm outline-none ${className}`.trim()} {...props}/>});
