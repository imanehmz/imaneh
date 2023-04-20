---
layout: post
title: Discord JS announcement slash command bot with image as argument:Slash commands with attachments
mathjax: true
---
There are two types of commands on discord js, message commands and slash commands. As of when I'm writing this article, Slash commands don't support embeds(images, attachments) as arguments, the issue has been raised [here](https://github.com/discord/discord-api-docs/issues/2322), adding this feature was requested, however, the team working on it had a problem in adding it due to some architecture problems.

## **How do we solve this problem?**

In this article, we'll proceed to create a channel that we'll call `#announce_content` where the command is going to be executed, it is going to be visible only by people having the role announcer, and it is going to receive the message content and the embeds within 15 seconds of receiving the slash command with its regular arguments.

## **Creating and organizing the bot:**

We'll create an `index.js`, `config.js`, `package.json`, and a `deploy_commands.js` files, a folder called `commands` where to put the files of our commands, in this case, we'll put the `announce.js` file inside. Copy the next code into your `package.json` file

```plaintext
{
    "dependencies": {
        "@discordjs/builders": "^0.9.0",
        "@discordjs/rest": "^0.1.0-canary.0",
        "discord": "^0.8.2",
        "discord-api-types": "^0.25.2",
        "discord.js": "^13.4.0",
        "js": "^0.1.0",
        "nvm": "^0.0.4",
        "pm2": "^5.1.2"
    }
}
```

and run

```plaintext
npm install
```

### **Getting the server, guild and client tokens**

Copy this code into your config.json file:

```plaintext
{
"DISCORD_TOKEN":"",
    "client_id":"",
    "guild_id":"",
}
```

follow the instructions on this [video](https://www.youtube.com/watch?v=JdpJiPlVeaU) to fill in these fields.

## Announcement command  

This command has one parameter which is the channel name: the channel where we want to announce, if you have many channels with the same name you can get the id of the channel as a parameter instead of the name.  
Copy this code into your `deploy_commands.js` file

```plaintext
const fs = require('fs');
const { REST } = require('@discordjs/rest');
const { Routes } = require('discord-api-types/v9');
const { client_id, guild_id, DISCORD_TOKEN } = require('./config.json');

const commands = [];
const commandFiles = fs.readdirSync('./commands').filter(file => file.endsWith('.js'));

for (const file of commandFiles) {
    const command = require(`./commands/${file}`);
    commands.push(command.data.toJSON());
}


const rest = new REST({ version: '9' }).setToken(DISCORD_TOKEN);

rest.put(Routes.applicationGuildCommands(client_id, guild_id), { body: commands })
    .then(() => console.log('Successfully registered application commands.'))
    .catch(console.error);
```

Copy this into your `index.js` file :

```plaintext
const fs = require('fs');
const { Client, Collection, Intents } = require('discord.js');
const { DISCORD_TOKEN } = require('./config.json');

const client = new Client({
    intents: [Intents.FLAGS.GUILDS, Intents.FLAGS.DIRECT_MESSAGES, Intents.FLAGS.GUILD_MESSAGES, Intents.FLAGS.GUILD_MESSAGE_REACTIONS]
});

client.once('ready', c => {
    console.log(`Ready! Logged in as ${c.user.tag}`);
});

client.on('interactionCreate', interaction => {
    console.log(`${interaction.user.tag} in #${interaction.channel.name} triggered an interaction.`);
});


client.commands = new Collection();
const commandFiles = fs.readdirSync('./commands').filter(file => file.endsWith('.js'));

for (const file of commandFiles) {
    const command = require(`./commands/${file}`);
    // Set a new item in the Collection
    // With the key as the command name and the value as the exported module
    client.commands.set(command.data.name, command);
}

client.on('interactionCreate', async interaction => {
    if (!interaction.isCommand()) ;
    const command = client.commands.get(interaction.commandName);

    if (!command) return;

    try {
        await command.execute(interaction,client);
    } catch (error) {
        console.error(error);
        await interaction.reply({ content: 'There was an error while executing this command!', ephemeral: true });
    }
});


// Login to Discord with your client's token
client.login(DISCORD_TOKEN);
```

It reads the commands from the commands folder, note that the command and the file name should carry the same name.  
Here's the full code for `Announce.js` file.

```plaintext
const { SlashCommandBuilder } = require('@discordjs/builders');
const { channel } = require('diagnostics_channel');
const { MessageEmbed } = require('discord.js');

const ephemeral = (msg) => {
    return {
        content: msg,
        ephemeral: true
    }
}

const wait = require('util').promisify(setTimeout);

module.exports = {
    name: "announce",
    description: "Announce a message in a specific channel",
    options: [{
        name: 'channel_name',
        description: 'Name of the channel where to announce',
        type: 'MENTIONABLE',
        required: true
    }],
    execute: async(client, interaction, args) => {
        try {
            if (!interaction.member.roles.cache.some(role => role.name === 'moderator')) {
                await interaction.reply(`You don't have access !`);
                return;
            }
            await interaction.deferReply(ephemeral("wait for the bot"));

            const channel_name = interaction.options.getString('channel_name');
            const announcement_channel = interaction.guild.channels.cache.get(channel_name.substring(2, channel_name.length - 1));

            if (interaction.channel.name !== "announce_content") {
                await interaction.reply(ephemeral("You're at the wrong channel!"));
            }

            await interaction.editReply(ephemeral("Send the message and the image in this channel now"));

            let recieved = false;
            const filter = m => m.author.id === interaction.member.id;
            const collector = interaction.channel.createMessageCollector({ filter, time: 15000 });

            collector.on('collect', (m) => {
                if (m) {
                    recieved = true;

                    if (m.attachments.size > 0) m.attachments.every((msgAttach) => {

                        const embedded_msg = new MessageEmbed().setTitle("Announcement").setDescription(m.content).setImage(msgAttach.url);

                        announcement_channel.send({
                            embeds: [embedded_msg]
                        });

                        interaction.editReply(ephemeral(`Your message has been successfully announced at ${channel_name}`));
                    });
                    else {
                        announcement_channel.send(m.content)
                    }
                    collector.stop();
                }
            });

            await wait(15000);
            if (!recieved) {
                interaction.editReply(ephemeral("I didn't receive your announcement :/"))
            }

        } catch (error) {
            console.log(error);
        }
    }


};
```

**We are using a collector that is going to send back the messages sent in that channel, either with embeds or without, it is going to be sent in 15000 ms**  
If you want it to be scheduled for a specific time, you can add the date and time of sending as parameters and use a Cron Job.  

# **BrainyBot2.0**  

This command has been realized for the open source bot BrainyBot2.0, made with ❤️ by the https://www.gdgalgiers.com/ Community.  
Check out the full source code on https://github.com/GDGAlgiers/BrainyBot2.0