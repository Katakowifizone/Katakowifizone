require('dotenv').config();
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');

const app = express();
app.use(express.json());
app.use(cors());

// Connexion à MongoDB
mongoose.connect(process.env.MONGO_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true
}).then(() => console.log('MongoDB connecté')).catch(err => console.error(err));

// Modèle Voucher
const VoucherSchema = new mongoose.Schema({
    code: String,
    duration: Number, // en heures
    used: { type: Boolean, default: false },
    createdAt: { type: Date, default: Date.now }
});
const Voucher = mongoose.model('Voucher', VoucherSchema);

// Générer un voucher
app.post('/api/vouchers', async (req, res) => {
    const { code, duration } = req.body;
    const newVoucher = new Voucher({ code, duration });
    await newVoucher.save();
    res.json({ message: 'Voucher créé', voucher: newVoucher });
});

// Vérifier un voucher
app.post('/api/vouchers/verify', async (req, res) => {
    const { code } = req.body;
    const voucher = await Voucher.findOne({ code, used: false });
    if (!voucher) return res.status(400).json({ message: 'Code invalide ou déjà utilisé' });
    
    voucher.used = true;
    await voucher.save();
    res.json({ message: 'Connexion réussie', duration: voucher.duration });
});

// Formules d'abonnement
const plans = [
    { name: '1 Heure', duration: 1, price: 2000 },
    { name: '24 Heures', duration: 24, price: 5000 },
    { name: '3 Jours', duration: 72, price: 10000 },
    { name: '1 Semaine', duration: 168, price: 25000 },
    { name: '1 Mois', duration: 720, price: 50000 }
];

app.get('/api/plans', (req, res) => {
    res.json(plans);
});

// Lancer le serveur
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Serveur en écoute sur le port ${PORT}`));

