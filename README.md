
const { Client, GatewayIntentBits } = require('discord.js');
const {
  joinVoiceChannel,
  createAudioPlayer,
  createAudioResource,
  NoSubscriberBehavior
} = require('@discordjs/voice');
require('dotenv').config();
const path = require('path');
const express = require('express');

// 🔁 سيرفر Express علشان UptimeRobot يقدر يزوره
const app = express();
app.get('/', (req, res) => res.send('Bot is running'));
app.listen(3000, () => console.log('🌐 Web server started'));

// 🎧 إعداد ديسكورد بوت
const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildVoiceStates
  ]
});

let connection, player;

client.once('ready', async () => {
  console.log(`✅ Logged in as ${client.user.tag}`);

  const guild = await client.guilds.fetch(process.env.GUILD_ID);
  const voiceChannel = guild.channels.cache.get(process.env.VOICE_CHANNEL_ID);

  if (!voiceChannel || voiceChannel.type !== 2) {
    console.error('❌ Invalid voice channel');
    return;
  }

  connection = joinVoiceChannel({
    channelId: process.env.VOICE_CHANNEL_ID,
    guildId: process.env.GUILD_ID,
    adapterCreator: guild.voiceAdapterCreator,
    selfDeaf: false
  });

  const resource = createAudioResource(path.join(__dirname, 'welcome.mp3'));
  player = createAudioPlayer({
    behaviors: {
      noSubscriber: NoSubscriberBehavior.Play
    }
  });

  connection.subscribe(player);
});

client.on('voiceStateUpdate', (oldState, newState) => {
  if (
    newState.channelId === process.env.VOICE_CHANNEL_ID &&
    oldState.channelId !== newState.channelId &&
    !newState.member.user.bot
  ) {
    const resource = createAudioResource(path.join(__dirname, 'welcome.mp3'));
    player.play(resource);
  }
});

client.login(process.env.TOKEN);

