[bt.txt](https://github.com/user-attachments/files/35439117/bt.txt)
const User = require('../users/user.js');



// --- AYARLAR ---

const MAX_COIN_LIMIT = 999999999999;



const FISH_TYPES = {

  3: ["👑 Efsanevi Ejderha Balığı", "🦈 Büyük Beyaz Köpekbalığı"],

  2: ["🦈 Çekiç Kafalı Köpekbalığı", "🐡 Balon Balığı"],

  1.5: ["🐟 Somon Balığı", "🐙 Ahtapot"],

  1: ["🐠 Hamsi", "🐠 İstavrit"],

  0: ["👢 Çizme", "🧴 Plastik Şişe", "🦀 Yosunlu Yengeç"]

};



const run = async (client, message, args) => {

  const miktar = parseInt(args[0]);



  if (isNaN(miktar) || miktar < 1 || miktar > 10000) {

    return message.reply('Lütfen **1 ile 10.000** arasında bir Coin miktarı girin! Örn: **.bt 100**');

  }



  let user = await User.findOne({ userID: message.author.id });

if (!user) user = new User({ userID: message.author.id });



  if (user.coin < miktar) {

    return message.reply(`Yetersiz bakiye! Mevcut Coin: \`${user.coin}\``);

  }



  // Şans Hesaplaması (1 - 100 arası)

  const rand = Math.random() * 100;

  let multiplier = 0;



  if (rand < 1) multiplier = 3;             // %1 İhtimal

  else if (rand < 16) multiplier = 2;        // %15 İhtimal (1 + 15)

  else if (rand < 36) multiplier = 1.5;      // %20 İhtimal (16 + 20)

  else if (rand < 61) multiplier = 1;        // %25 İhtimal (36 + 25)

  else multiplier = 0;                       // Kalan %39 İhtimal



  const kazanilan = Math.floor(miktar * multiplier);

  const netDegisim = kazanilan - miktar;



  // Bakiye Güncelleme

  user.coin = Math.min(MAX_COIN_LIMIT, Math.max(0, user.coin + netDegisim));

  await user.save();



  // Rastgele İtem Seçimi

  const items = FISH_TYPES[multiplier];

  const item = items[Math.floor(Math.random() * items.length)];



  // Durum Mesajı

  let durumText = "";

  if (multiplier > 1) durumText = `🔥 **Tebrikler! ${multiplier}x Katladın!** (+\`${kazanilan
miktar}\` Coin)`;

  else if (multiplier === 1) durumText = `⚖️ **Paranı Geri Aldın!** (0 Coin)`;

  else durumText = `😢 **Kaybettin, Paran Yandı!** (-\`${miktar}\` Coin)`;



  return message.reply(

    `🎣 Olta atıldı ve... **${item}** yakaladın!\n` +

    `${durumText}\n` +

    `🪙 **Güncel Coin:** \`${user.coin}\``

  );

};



module.exports = {

  config: {

    enabled: true,

    guildOnly: true,

    name: "balıktut",

    aliases: ["baliktut", "balık", "balik", "bt"],

    cooldown: 15

  },

  run: run

};
