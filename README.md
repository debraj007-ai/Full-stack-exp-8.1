import express from "express";
import mongoose from "mongoose";
import cors from "cors";

// --- Connect to MongoDB ---
mongoose.connect("mongodb://localhost:27017/productdb", {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log(" MongoDB connected"))
.catch(err => console.error(" Connection error:", err));

// --- Define Product Schema ---
const productSchema = new mongoose.Schema({
  name: { type: String, required: true },
  price: { type: Number, required: true },
  category: String,
  inStock: { type: Boolean, default: true },
}, { timestamps: true });

const Product = mongoose.model("Product", productSchema);

// --- Setup Express App ---
const app = express();
app.use(express.json());
app.use(cors());

// --- CRUD Routes ---

//  Create
app.post("/api/products", async (req, res) => {
  try {
    const product = new Product(req.body);
    res.status(201).json(await product.save());
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

//  Read All
app.get("/api/products", async (req, res) => {
  res.json(await Product.find());
});

//  Read One
app.get("/api/products/:id", async (req, res) => {
  try {
    const product = await Product.findById(req.params.id);
    product ? res.json(product) : res.status(404).json({ message: "Not found" });
  } catch {
    res.status(400).json({ message: "Invalid ID" });
  }
});

//  Update
app.put("/api/products/:id", async (req, res) => {
  try {
    const updated = await Product.findByIdAndUpdate(req.params.id, req.body, { new: true });
    updated ? res.json(updated) : res.status(404).json({ message: "Not found" });
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

// Delete
app.delete("/api/products/:id", async (req, res) => {
  try {
    const deleted = await Product.findByIdAndDelete(req.params.id);
    deleted ? res.json({ message: "Deleted successfully" }) : res.status(404).json({ message: "Not found" });
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
});

// --- Start Server ---
const PORT = 5000;
app.listen(PORT, () => console.log(` Server running on http://localhost:${PORT}`));
