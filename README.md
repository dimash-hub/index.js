import express from "express";
import TelegramBot from "node-telegram-bot-api";

const app = express();
const PORT = process.env.PORT || 3000;

app.get("/", (_req, res) => res.send("JEETS Whale Bot is alive on Render!"));

const TELEGRAM_TOKEN = process.env.TELEGRAM_TOKEN;
const CHAT_ID = process.env.CHAT_ID;
const bot = new TelegramBot(TELEGRAM_TOKEN, { polling: true });

function sendWhaleSignal(msg) {
  if (!TELEGRAM_TOKEN || !CHAT_ID) return;
  bot.sendMessage(CHAT_ID, msg);
}

setInterval(() => {
  sendWhaleSignal("🐋 Тест: бот жив и JEETS сигнал работает!");
}, 30000);

app.listen(PORT, () => console.log(`Server listening on port ${PORT}`));
// === ВРЕМЕННЫЙ: список всех SPL-токенов кита (помогает найти, видит ли бот JEETS вообще)
app.get("/debug/tokens", async (_req, res) => {
  try {
    const body = {
      jsonrpc: "2.0",
      id: 1,
      method: "getParsedTokenAccountsByOwner",
      params: [
        process.env.WHALE_ADDRESS,
        { programId: "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA" },
        { encoding: "jsonParsed" }
      ],
    };
    const { data } = await axios.post(process.env.SOLANA_RPC || "https://api.mainnet-beta.solana.com", body, { timeout: 15000 });
    const accounts = data?.result?.value || [];
    const list = accounts.map(acc => {
      const info = acc?.account?.data?.parsed?.info;
      return {
        mint: info?.mint,
        uiAmount: info?.tokenAmount?.uiAmount
      };
    });
    res.json({ whale: process.env.WHALE_ADDRESS, count: list.length, tokens: list });
  } catch (e) {
    res.status(500).json({ error: String(e?.message || e) });
  }
});

// === ВРЕМЕННЫЙ: статус (баланс JEETS + цена)
app.get("/status", async (_req, res) => {
  try {
    const balance = await getJeetsBalanceUi(process.env.WHALE_ADDRESS, process.env.JEETS_TOKEN_ADDRESS);
    const price = await getJeetsPriceUsd(process.env.JEETS_TOKEN_ADDRESS);
    res.json({
      whale: process.env.WHALE_ADDRESS,
      jeetsMint: process.env.JEETS_TOKEN_ADDRESS,
      balance,
      price,
      usdValue: balance * price,
      lastCheck: new Date().toISOString()
    });
  } catch (e) {
    res.status(500).json({ error: String(e?.message || e) });
  }
});
