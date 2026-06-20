const { Client, GatewayIntentBits } = require("discord.js");
const fs = require("fs");

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMessages,
    GatewayIntentBits.MessageContent
  ]
});

const membersFile = "./members.json";
const counterFile = "./counter.json";

function load(file) {
  return JSON.parse(fs.readFileSync(file, "utf8"));
}

function save(file, data) {
  fs.writeFileSync(file, JSON.stringify(data, null, 2));
}

const waiting = {};

client.on("messageCreate", async (msg) => {
  if (msg.author.bot) return;

  const id = msg.author.id;

  if (msg.content === "/会員登録") {
    waiting[id] = true;
    return msg.reply("好きなポケモンを入力してね");
  }

  if (waiting[id]) {
    const members = load(membersFile);
    const counter = load(counterFile);

    const num = counter.nextNumber;

    members[id] = {
      memberNumber: num,
      favoritePokemon: msg.content,
      joinDate: new Date().toISOString().split("T")[0],
      username: msg.author.username
    };

    counter.nextNumber++;

    save(membersFile, members);
    save(counterFile, counter);

    delete waiting[id];

    return msg.reply(`登録完了！会員番号：${String(num).padStart(4,"0")}`);
  }

  if (msg.content === "/会員証") {
    const members = load(membersFile);
    const data = members[id];

    if (!data) return msg.reply("まだ登録されてないよ");

    return msg.reply(
      `🖤 こわおねランド会員証 🖤\n` +
      `名前：${data.username}\n` +
      `会員番号：${String(data.memberNumber).padStart(4,"0")}\n` +
      `好きなポケモン：${data.favoritePokemon}\n` +
      `入会日：${data.joinDate}`
    );
  }
});

client.login(process.env.DISCORD_BOT_TOKEN);
