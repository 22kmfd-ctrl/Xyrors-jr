const { Client, GatewayIntentBits, EmbedBuilder, ActionRowBuilder, ButtonBuilder, ButtonStyle } = require('discord.js');
const fetch = require('node-fetch');
const express = require('express');

const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
    res.send('¡Bot activo y funcionando 24/7!');
});

app.listen(PORT, () => {
    console.log(`Servidor web corriendo en el puerto ${PORT}`);
});

const client = new Client({
    intents: [
        GatewayIntentBits.Guilds,
        GatewayIntentBits.GuildMessages,
        GatewayIntentBits.MessageContent
    ]
});

const DOMINIOS_PROHIBIDOS = [
    'xvideos.com', 'pornhub.com', 'xnxx.com', 'redtube.com', 
    'youporn.com', 'eporner.com', 'spankbang.com', 'camcaps.com',
    'onlyfans.com', 'fansly.com', 'chaturbate.com', 'livejasmin.com'
];

client.once('ready', () => {
    console.log(`¡Bot conectado como ${client.user.tag}!`);
});

client.on('messageCreate', async message => {
    if (message.author.bot || !message.content) return;

    const text = message.content;

    if (text.startsWith('!anime')) {
        const animeEmbed = new EmbedBuilder()
            .setColor('#3b82f6')
            .setTitle('🌸 Directorio de Anime y Manga')
            .addFields(
                { name: '📺 Ver Anime', value: '[Crunchyroll](https://www.crunchyroll.com)', inline: false },
                { name: '📚 Leer Manga', value: '[MangaPlus](https://mangaplus.shueisha.co.jp) | [Webtoon](https://www.webtoon.com)', inline: false }
            )
            .setFooter({ text: `Solicitado por ${message.author.username}` });
        return message.reply({ embeds: [animeEmbed] });
    }

    if (text.startsWith('!buscar ')) {
        const query = text.replace('!buscar ', '').trim();
        if (!query) return message.reply('❌ Escribe algo para buscar.');
        try {
            const searchUrl = `https://api.duckduckgo.com/?q=${encodeURIComponent(query)}&format=json&no_html=1&skip_disambig=1`;
            const res = await fetch(searchUrl);
            const searchData = await res.json();
            let resultText = searchData.AbstractText || searchData.Heading;
            let resultUrl = searchData.AbstractURL;

            const searchEmbed = new EmbedBuilder()
                .setColor('#10b981')
                .setTitle(`🔍 Resultados para: "${query}"`);

            if (resultText) {
                searchEmbed.setDescription(resultText);
                if (resultUrl) searchEmbed.addFields({ name: 'Fuente', value: `[Enlace](${resultUrl})` });
            } else {
                searchEmbed.setDescription(`Sin respuesta directa.`);
            }
            return message.reply({ embeds: [searchEmbed] });
        } catch (e) {
            return message.reply('❌ Error en la búsqueda.');
        }
    }

    const urlMatch = text.match(/(https?:\/\/[^\s]+)/);
    if (!urlMatch) return;
    const targetUrl = urlMatch[0];

    const esProhibido = DOMINIOS_PROHIBIDOS.some(dominio => targetUrl.toLowerCase().includes(dominio));
    if (esProhibido) {
        try {
            await message.delete();
            const advertencia = await message.channel.send(`⚠️ ${message.author}, contenido prohibido.`);
            setTimeout(() => advertencia.delete().catch(() => {}), 5000);
        } catch (error) {}
        return;
    }

    const isTikTok = targetUrl.includes('tiktok.com');
    const isInstagram = targetUrl.includes('instagram.com');
    const isYouTube = targetUrl.includes('youtube.com') || targetUrl.includes('youtu.be');
    const isFacebook = targetUrl.includes('facebook.com') || targetUrl.includes('fb.watch');

    if (isTikTok || isInstagram || isYouTube || isFacebook) {
        const processingMsg = await message.reply('⏳ Procesando video...');

        try {
            const apiURL = `https://tikwm.com/api/?url=${encodeURIComponent(targetUrl)}`;
            const response = await fetch(apiURL);
            const json = await response.json();

            let videoUrl = null;
            if (json && json.data && json.data.play) {
                videoUrl = json.data.play;
            }

            if (videoUrl) {
                await processingMsg.delete();
                await message.delete().catch(() => {});

                const row = new ActionRowBuilder()
                    .addComponents(
                        new ButtonBuilder()
                            .setLabel('🔗 Ver Original')
                            .setStyle(ButtonStyle.Link)
                            .setURL(targetUrl)
                    );

                return message.channel.send({
                    content: `👤 **${message.author}** envió un video:\n${videoUrl}`,
                    components: [row]
                });
            } else {
                await processingMsg.delete();
                await message.delete().catch(() => {});

                const embed = new EmbedBuilder()
                    .setColor('#FF0000')
                    .setTitle('🎬 Enlace Multimedia Detectado')
                    .setDescription(`Solicitado por ${message.author}`);

                const row = new ActionRowBuilder()
                    .addComponents(
                        new ButtonBuilder()
                            .setLabel('🔗 Ver Contenido Original')
                            .setStyle(ButtonStyle.Link)
                            .setURL(targetUrl)
                    );

                return message.channel.send({
                    content: `👤 **${message.author}** compartió un enlace:`,
                    embeds: [embed],
                    components: [row]
                });
            }

        } catch (error) {
            console.error(error);
            await processingMsg.edit('❌ Ocurrió un error al procesar el enlace.');
        }
    }
});

client.login(process.env.TOKEN);
