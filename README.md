# Nikhil-website-
Make a website 
{
  "name": "nikhil-property-website",
  "version": "1.0.0",
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  },
  "dependencies": {
    "next": "^12.0.0",
    "react": "^17.0.0",
    "react-dom": "^17.0.0",
    "mongoose": "^6.0.0",
    "multer": "^1.4.0",
    "nodemailer": "^6.7.0",
    "tailwindcss": "^3.0.0",
    "autoprefixer": "^10.4.0",
    "postcss": "^8.4.0",
    "react-floating-whatsapp": "^1.0.0"
  }
}
module.exports = {
  reactStrictMode: true,
};import mongoose from 'mongoose';

const connectDB = async () => {
  if (mongoose.connections[0].readyState) return;
  await mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost:27017/propertydb', {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  });
};

export default connectDB;import mongoose from 'mongoose';

const propertySchema = new mongoose.Schema({
  title: String,
  description: String,
  price: Number,
  area: Number,
  location: String,
  type: { type: String, enum: ['Residential', 'Commercial'] },
  status: { type: String, enum: ['For Sale', 'Sold'] },
  images: [String],
  createdAt: { type: Date, default: Date.now },
});

export default mongoose.models.Property || mongoose.model('Property', propertySchema);import connectDB from '../../lib/db';
import Property from '../../models/Property';
import multer from 'multer';
import { promisify } from 'util';

const upload = multer({ dest: 'uploads/' });

export const config = {
  api: {
    bodyParser: false,
  },
};

export default async function handler(req, res) {
  await connectDB();

  if (req.method === 'GET') {
    const properties = await Property.find({});
    res.status(200).json(properties);
  } else if (req.method === 'POST') {
    upload.array('images')(req, res, async (err) => {
      if (err) return res.status(500).json({ error: err.message });
      const { title, description, price, area, location, type, status } = req.body;
      const images = req.files.map(file => `/uploads/${file.filename}`);
      const property = new Property({ title, description, price, area, location, type, status, images });
      await property.save();
      res.status(201).json(property);
    });
  }
}import nodemailer from 'nodemailer';

export default async function handler(req, res) {
  if (req.method === 'POST') {
    const { name, email, phone, message } = req.body;

    const transporter = nodemailer.createTransporter({
      service: 'gmail',
      auth: {
        user: 'your-email@gmail.com', // अपना Gmail डालें (जैसे nikhilthakre390@gmail.com)
        pass: 'your-app-password', // Gmail App Password जनरेट करें
      },
    });

    const mailOptions = {
      from: email,
      to: 'nikhilthakre390@gmail.com',
      subject: 'New Contact Inquiry from Nikhil Thakre Website',
      text: `Name: ${name}\nEmail: ${email}\nPhone: +91 9022138468\nMessage: ${message}`,
    };

    try {
      await transporter.sendMail(mailOptions);
      res.status(200).json({ message: 'Email sent successfully' });
    } catch (error) {
      res.status(500).json({ error: 'Failed to send email' });
    }
  }
}import { useState, useEffect } from 'react';
import Header from '../components/Header';
import Footer from '../components/Footer';
import FloatingWhatsApp from 'react-floating-whatsapp';

export default function Home() {
  const [properties, setProperties] = useState([]);
  const [search, setSearch] = useState({ location: '', type: '', budget: '' });

  useEffect(() => {
    fetch('/api/properties')
      .then(res => res.json())
      .then(data => setProperties(data));
  }, []);

  const handleSearch = () => {
    // सिम्पल सर्च (आप इसे एडवांस कर सकते हैं)
    alert(`Searching for ${search.type} in ${search.location} within ${search.budget}`);
  };

  return (
    <div>
      <Header />
      <section className="hero bg-gradient-to-r from-blue-900 to-yellow-500 text-white py-20 text-center">
        <h1 className="text-5xl font-bold mb-4">Discover Your Perfect Property</h1>
        <p className="text-xl mb-8">Expert Buying & Selling of Residential & Commercial Properties</p>
        <div className="search-bar flex justify-center gap-4 flex-wrap">
          <select value={search.location} onChange={(e) => setSearch({...search, location: e.target.value})} className="p-2 rounded">
            <option>Location</option>
            <option>Mumbai</option>
            <option>Pune</option>
          </select>
          <select value={search.type} onChange={(e) => setSearch({...search, type: e.target.value})} className="p-2 rounded">
            <option>Property Type</option>
            <option>Residential</option>
            <option>Commercial</option>
          </select>
          <select value={search.budget} onChange={(e) => setSearch({...search, budget: e.target.value})} className="p-2 rounded">
            <option>Budget</option>
            <option>₹50L - ₹1Cr</option>
            <option>₹1Cr - ₹5Cr</option>
          </select>
          <button onClick={handleSearch} className="bg-yellow-500 px-6 py-2 rounded font-bold">Search Now</button>
        </div>
      </section>
      <section className="properties py-10 bg-gray-100">
        <div className="container mx-auto px-4">
          <h2 className="text-3xl font-bold text-center mb-8">Featured Properties</h2>
          <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
            {properties.map(prop => (
              <div key={prop._id} className="bg-white p-4 shadow-lg rounded-lg">
                <img src={prop.images[0] || 'https://via.placeholder.com/300x200?text=No+Image'} alt={prop.title} className="w-full h-40 object-cover rounded" />
                <h3 className="text-xl font-semibold mt-4">{prop.title}</h3>
                <p className="text-gray-600">₹{prop.price} | {prop.area} sq. ft. | {prop.location}</p>
                <span className={`inline-block px-3 py-1 rounded text-white ${prop.status === 'Sold' ? 'bg-red-500' : 'bg-green-500'}`}>{prop.status}</span>
                <button className="mt-4 bg-blue-600 text-white px-4 py-2 rounded">View Details</button>
              </div>
            ))}
          </div>
        </div>
      </section>
      <section className="about py-10 text-center">
        <h2 className="text-3xl font-bold mb-4">Why Choose Us?</h2>
        <ul className="list-none">
          <li className="mb-2">Trusted Expertise in Real Estate</li>
          <li className="mb-2">Wide Network Across India</li>
          <li>Personalized Property Solutions</li>
        </ul>
      </section>
      <section className="contact py-10 bg-blue-900 text-white">
        <div className="container mx-auto px-4 text-center">
          <h2 className="text-3xl font-bold mb-8">Contact Us</h2>
          <form onSubmit={async (e) => {
            e.preventDefault();
            const formData = new FormData(e.target);
            const data = Object.fromEntries(formData);
            const res = await fetch('/api/contact', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(data) });
            alert(res.ok ? 'Message sent!' : 'Error sending message');
          }} className="max-w-md mx-auto">
            <input name="name" type="text" placeholder="Name" required className="w-full p-2 mb-4 rounded" />
            <input name="email" type="email" placeholder="Email" required className="w-full p-2 mb-4 rounded" />
            <input name="phone" type="tel" placeholder="Phone" required className="w-full p-2 mb-4 rounded" />
            <textarea name="message" placeholder="Message" required className="w-full p-2 mb-4 rounded"></textarea>
            <button type="submit" className="bg-yellow-500 px-6 py-2 rounded font-bold">Send Message</button>
          </form>
          <p className="mt-4">Contact: +91 9022138468 | Email: nikhilthakre390@gmail.com</p>
        </div>
      </section>
      <FloatingWhatsApp phoneNumber="919022138468" message="Hi, I'm interested in a property!" />
      <Footer />
    </div>
  );
}import { useState } from 'react';

export default function Admin() {
  const [form, setForm] = useState({ title: '', description: '', price: '', area: '', location: '', type: '', status: '' });

  const handleSubmit = async (e) => {
    e.preventDefault();
    const formData = new FormData();
    Object.keys(form).forEach(key => formData.append(key, form[key]));
    // इमेजेस ऐड करें (फाइल इनपुट से)
    const res = await fetch('/api/properties', { method: 'POST', body: formData });
    if (res.ok) alert('Property added successfully!');
  };

  return (
    <div className="p-10 bg-gray-100 min-h-screen">
      <h1 className="text-3xl font-bold mb-8">Admin Dashboard - Upload Property</h1>
      <form onSubmit={handleSubmit} className="max-w-md mx-auto bg-white p-6 rounded shadow">
        <input placeholder="Title" onChange={(e) => setForm({...form, title: e.target.value})} className="w-full p-2 mb-4 border rounded" />
        <textarea placeholder="Description" onChange={(e) => setForm({...form, description: e.target.value})} className="w-full p-2 mb-4 border rounded"></textarea>
        <input placeholder="Price" type="number" onChange={(e) => setForm({...form, price: e.target.value})} className="w-full p-2 mb-4 border rounded" />
        <input placeholder="Area (sq. ft.)" type="number" onChange={(e) => setForm({...form, area: e.target.value})} className="w-full p-2 mb-4 border rounded" />
        <input placeholder="Location" onChange={(e) => setForm({...form, location: e.target.value})} className="w-full p-2 mb-4 border rounded" />
        <select onChange={(e) => setForm({...form, type: e.target.value})} className="w-full p-2 mb-4 border rounded">
          <option>Type</option>
          <option>Residential</option>
          <option>Commercial</option>
        </select>
        <select onChange={(e) => setForm({...form, status: e.target.value})} className="w-full p-2 mb-4 border rounded">
          <option>Status</option>
          <option>For Sale</option>
          <option>Sold</option>
        </select>
        <input type="file" multiple name="images" className="w-full p-2 mb-4" />
        <button type="submit" className="bg-blue-600 text-white px-6 py-2 rounded font-bold">Upload Property</button>
      </form>
    </div>
  );
}export default function Header() {
  return (
    <header className="bg-blue-900 text-white p-4 flex justify-between items-center">
      <h1 className="text-2xl font-bold">Nikhil Thakre - Property Solutions</h1>
      <nav className="space-x-4">
        <a href="/" className="hover:underline">Home</a>
        <a href="/properties" className="hover:underline">Properties</a>
        <a href="/admin" className="hover:underline">Admin</a>
      </nav>
    </header>
  );
}export default function Footer() {
  return (
    <footer className="bg-blue-900 text-white text-center p-4">
      <p>Contact: +91 9022138468 | Email: nikhilthakre390@gmail.com</p>
      <p>&copy; 2023 Nikhil Thakre - Property Solutions. All rights reserved.</p>
    </footer>
  );
}@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  font-family: 'Roboto', sans-serif;
}
