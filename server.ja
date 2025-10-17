const express = require('express');
const fetch = require('node-fetch');
const cheerio = require('cheerio');
const cors = require('cors');

const app = express();
app.use(cors());
app.use(express.json());
app.use(express.static(__dirname));

app.get('/download', async (req, res) => {
  try {
    const { url } = req.query;
    if(!url) return res.status(400).json({error:'Missing URL parameter'});
    
    const response = await fetch(url, { headers: { 'User-Agent':'Mozilla/5.0' }});
    const html = await response.text();
    
    const $ = cheerio.load(html);
    
    // Try video first
    let videoUrl = $('meta[property="og:video"]').attr('content');
    if(videoUrl) return res.json({ type:'video', url: videoUrl });
    
    // Try image
    let imageUrl = $('meta[property="og:image"]').attr('content');
    if(imageUrl) return res.json({ type:'image', url: imageUrl });
    
    // Fallback: check JSON embedded for video_url (reels, IGTV)
    const scripts = $('script[type="text/javascript"]').toArray();
    for(const script of scripts) {
      const htmlContent = $(script).html();
      if(htmlContent && htmlContent.includes('video_url')) {
        const match = htmlContent.match(/"video_url":"(https:[^"]+)"/);
        if(match && match[1]) {
          const video = match[1].replace(/\\u0026/g,'&').replace(/\\/g,'');
          return res.json({ type:'video', url: video });
        }
      }
    }
    
    return res.status(404).json({ error:'Cannot fetch content. Maybe private post.'});
  } catch(err) {
    console.error(err);
    return res.status(500).json({ error:'Internal Server Error'});
  }
});

const PORT = 3000;
app.listen(PORT, ()=>console.log('✅ Server running at http://localhost:'+PORT));
