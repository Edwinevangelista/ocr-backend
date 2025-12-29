const express = require('express');
const cors = require('cors');
const Anthropic = require('@anthropic-ai/sdk').default;
const pdfParse = require('pdf-parse');

const app = express();
const PORT = process.env.PORT || 3001;

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

app.use(cors());
app.use(express.json({ limit: '10mb' }));

app.get('/', (req, res) => {
  res.json({ status: 'OCR API Running' });
});

app.post('/scan', async (req, res) => {
  try {
    const { file, fileType } = req.body;

    if (!file) {
      return res.status(400).json({ error: 'No file provided' });
    }

    let base64Data = file;
    let mediaType = 'image/jpeg';

    if (fileType === 'application/pdf') {
      const pdfBuffer = Buffer.from(file.split(',')[1] || file, 'base64');
      const data = await pdfParse(pdfBuffer);
      
      const message = await anthropic.messages.create({
        model: 'claude-sonnet-4-20250514',
        max_tokens: 4096,
        messages: [{
          role: 'user',
          content: `Analiza este estado de cuenta y extrae TODAS las transacciones.

${data.text}

Devuelve SOLO JSON:
{
  "transacciones": [
    {
      "fecha": "2025-10-15",
      "descripcion": "WALMART",
      "monto": -45.67,
      "categoria": "Supermercado",
      "tipo": "gasto"
    }
  ],
  "resumen": {
    "total_ingresos": 0,
    "total_gastos": 0,
    "cantidad_transacciones": 0
  }
}`
        }]
      });

      const responseText = message.content[0].text;
      const jsonMatch = responseText.match(/\{[\s\S]*\}/);
      const result = JSON.parse(jsonMatch[0]);
      return res.json(result);

    } else {
      if (file.includes('base64,')) {
        base64Data = file.split(',')[1];
      }

      if (file.startsWith('data:image/png')) mediaType = 'image/png';
      else if (file.startsWith('data:image/webp')) mediaType = 'image/webp';

      const message = await anthropic.messages.create({
        model: 'claude-sonnet-4-20250514',
        max_tokens: 4096,
        messages: [{
          role: 'user',
          content: [
            {
              type: 'image',
              source: { type: 'base64', media_type: mediaType, data: base64Data }
            },
            {
              type: 'text',
              text: `Extrae transacciones de esta imagen. Devuelve SOLO JSON con formato: {"transacciones": [...], "resumen": {...}}`
            }
          ]
        }]
      });

      const responseText = message.content[0].text;
      const jsonMatch = responseText.match(/\{[\s\S]*\}/);
      const result = JSON.parse(jsonMatch[0]);
      return res.json(result);
    }

  } catch (error) {
    console.error('Error:', error);
    res.status(500).json({ error: error.message });
  }
});


// ===== AUTH PROXY ENDPOINTS =====

// Signup proxy
app.post('/auth/signup', async (req, res) => {
  try {
    const { email, password } = req.body;
    
    const response = await fetch('https://loluismsoljdsoksuiei.supabase.co/auth/v1/signup', {
      method: 'POST',
      headers: {
        'apikey': 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImxvbHVpc21zb2xqZHNva3N1aWVpIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NjY5NjU5NjksImV4cCI6MjA4MjU0MTk2OX0.VVw4acoZZayYs7ONe88-XjUXxXmFcyjPy2hJuq_-rDs',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ email, password })
    });
    
    const data = await response.json();
    res.json(data);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Login proxy
app.post('/auth/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    
    const response = await fetch('https://loluismsoljdsoksuiei.supabase.co/auth/v1/token?grant_type=password', {
      method: 'POST',
      headers: {
        'apikey': 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImxvbHVpc21zb2xqZHNva3N1aWVpIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NjY5NjU5NjksImV4cCI6MjA4MjU0MTk2OX0.VVw4acoZZayYs7ONe88-XjUXxXmFcyjPy2hJuq_-rDs',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ email, password })
    });
    
    const data = await response.json();
    res.json(data);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
