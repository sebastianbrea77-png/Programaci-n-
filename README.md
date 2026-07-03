# Programaci-n-
Proyectos de la asignatura de programación 
calculadora:
```react
import React, { useState, useMemo, useEffect } from 'react';
import { 
  Package, 
  Plus, 
  Search, 
  AlertTriangle, 
  TrendingUp, 
  DollarSign, 
  Trash2, 
  Edit3, 
  ArrowUpRight, 
  ArrowDownLeft,
  ChevronRight,
  Filter,
  X
} from 'lucide-react';

const App = () => {
  // --- State Management ---
  const [inventory, setInventory] = useState([
    { id: 1, name: 'Laptop Pro 15"', category: 'Electrónica', price: 1200, stock: 15, minStock: 5 },
    { id: 2, name: 'Monitor 4K', category: 'Electrónica', price: 400, stock: 3, minStock: 10 },
    { id: 3, name: 'Silla Ergonómica', category: 'Mobiliario', price: 250, stock: 25, minStock: 8 },
    { id: 4, name: 'Teclado Mecánico', category: 'Accesorios', price: 80, stock: 40, minStock: 15 },
    { id: 5, name: 'Escritorio de Madera', category: 'Mobiliario', price: 300, stock: 2, minStock: 5 },
  ]);

  const [isModalOpen, setIsModalOpen] = useState(false);
  const [searchTerm, setSearchTerm] = useState('');
  const [filterCategory, setFilterCategory] = useState('Todos');
  const [editingProduct, setEditingProduct] = useState(null);

  // --- Derived Data (Analytics) ---
  const stats = useMemo(() => {
    const totalItems = inventory.reduce((acc, item) => acc + item.stock, 0);
    const totalValue = inventory.reduce((acc, item) => acc + (item.price * item.stock), 0);
    const lowStockItems = inventory.filter(item => item.stock <= item.minStock).length;
    return { totalItems, totalValue, lowStockItems };
  }, [inventory]);

  const categories = ['Todos', ...new Set(inventory.map(item => item.category))];

  const filteredInventory = inventory.filter(item => {
    const matchesSearch = item.name.toLowerCase().includes(searchTerm.toLowerCase());
    const matchesCategory = filterCategory === 'Todos' || item.category === filterCategory;
    return matchesSearch && matchesCategory;
  });

  // --- Actions ---
  const handleSaveProduct = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const productData = {
      name: formData.get('name'),
      category: formData.get('category'),
      price: parseFloat(formData.get('price')),
      stock: parseInt(formData.get('stock')),
      minStock: parseInt(formData.get('minStock')),
    };

    if (editingProduct) {
      setInventory(prev => prev.map(p => p.id === editingProduct.id ? { ...productData, id: p.id } : p));
    } else {
      setInventory(prev => [...prev, { ...productData, id: Date.now() }]);
    }
    closeModal();
  };

  const deleteProduct = (id) => {
    if (window.confirm('¿Estás seguro de eliminar este producto?')) {
      setInventory(prev => prev.filter(p => p.id !== id));
    }
  };

  const adjustStock = (id, amount) => {
    setInventory(prev => prev.map(p => {
      if (p.id === id) {
        const newStock = Math.max(0, p.stock + amount);
        return { ...p, stock: newStock };
      }
      return p;
    }));
  };

  const openModal = (product = null) => {
    setEditingProduct(product);
    setIsModalOpen(true);
  };

  const closeModal = () => {
    setEditingProduct(null);
    setIsModalOpen(false);
  };

  return (
    <div className="min-h-screen bg-slate-50 text-slate-900 font-sans p-4 md:p-8">
      <div className="max-w-7xl mx-auto">
        
        {/* Header */}
        <header className="flex flex-col md:flex-row md:items-center justify-between mb-8 gap-4">
          <div>
            <h1 className="text-3xl font-bold text-slate-800 tracking-tight">Panel de Inventario</h1>
            <p className="text-slate-500">Gestiona tus productos y existencias en tiempo real</p>
          </div>
          <button 
            onClick={() => openModal()}
            className="flex items-center justify-center gap-2 bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-3 rounded-xl font-semibold transition-all shadow-lg shadow-indigo-200 active:scale-95"
          >
            <Plus size={20} />
            Nuevo Producto
          </button>
        </header>

        {/* Stats Grid */}
        <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
          <StatCard 
            title="Total Productos" 
            value={stats.totalItems} 
            icon={<Package className="text-blue-600" />} 
            color="bg-blue-50"
          />
          <StatCard 
            title="Valor Inventario" 
            value={`$${stats.totalValue.toLocaleString()}`} 
            icon={<DollarSign className="text-emerald-600" />} 
            color="bg-emerald-50"
          />
          <StatCard 
            title="Bajo Stock" 
            value={stats.lowStockItems} 
            icon={<AlertTriangle className="text-amber-600" />} 
            color="bg-amber-50"
            alert={stats.lowStockItems > 0}
          />
        </div>

        {/* Filters & Search */}
        <div className="bg-white p-4 rounded-2xl shadow-sm border border-slate-200 mb-6 flex flex-col md:flex-row gap-4 items-center">
          <div className="relative w-full md:w-96">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400" size={18} />
            <input 
              type="text" 
              placeholder="Buscar producto..." 
              className="w-full pl-10 pr-4 py-2 rounded-lg border border-slate-200 focus:outline-none focus:ring-2 focus:ring-indigo-500 transition-all"
              value={searchTerm}
              onChange={(e) => setSearchTerm(e.target.value)}
            />
          </div>
          <div className="flex items-center gap-2 w-full md:w-auto">
            <Filter size={18} className="text-slate-400" />
            <select 
              className="bg-slate-50 border border-slate-200 rounded-lg px-3 py-2 outline-none focus:ring-2 focus:ring-indigo-500 transition-all"
              value={filterCategory}
              onChange={(e) => setFilterCategory(e.target.value)}
            >
              {categories.map(cat => (
                <option key={cat} value={cat}>{cat}</option>
              ))}
            </select>
          </div>
        </div>

        {/* Table/List */}
        <div className="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
          <div className="overflow-x-auto">
            <table className="w-full text-left">
              <thead className="bg-slate-50 border-b border-slate-200">
                <tr>
                  <th className="px-6 py-4 text-sm font-semibold text-slate-600 uppercase">Producto</th>
                  <th className="px-6 py-4 text-sm font-semibold text-slate-600 uppercase">Categoría</th>
                  <th className="px-6 py-4 text-sm font-semibold text-slate-600 uppercase">Precio</th>
                  <th className="px-6 py-4 text-sm font-semibold text-slate-600 uppercase text-center">Stock</th>
                  <th className="px-6 py-4 text-sm font-semibold text-slate-600 uppercase text-right">Acciones</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-slate-100">
                {filteredInventory.map((item) => (
                  <tr key={item.id} className="hover:bg-slate-50/50 transition-colors">
                    <td className="px-6 py-4">
                      <div className="font-medium text-slate-800">{item.name}</div>
                      {item.stock <= item.minStock && (
                        <span className="text-[10px] bg-amber-100 text-amber-700 px-2 py-0.5 rounded-full font-bold uppercase tracking-wider">Stock Bajo</span>
                      )}
                    </td>
                    <td className="px-6 py-4 text-slate-600">
                      <span className="bg-slate-100 px-3 py-1 rounded-full text-xs font-medium">
                        {item.category}
                      </span>
                    </td>
                    <td className="px-6 py-4 text-slate-700 font-medium">
                      ${item.price.toFixed(2)}
                    </td>
                    <td className="px-6 py-4">
                      <div className="flex items-center justify-center gap-3">
                        <button 
                          onClick={() => adjustStock(item.id, -1)}
                          className="p-1 rounded-md hover:bg-red-50 text-red-500 border border-transparent hover:border-red-100 transition-all"
                        >
                          <ArrowDownLeft size={16} />
                        </button>
                        <span className={`w-10 text-center font-bold ${item.stock <= item.minStock ? 'text-amber-600' : 'text-slate-800'}`}>
                          {item.stock}
                        </span>
                        <button 
                          onClick={() => adjustStock(item.id, 1)}
                          className="p-1 rounded-md hover:bg-emerald-50 text-emerald-500 border border-transparent hover:border-emerald-100 transition-all"
                        >
                          <ArrowUpRight size={16} />
                        </button>
                      </div>
                    </td>
                    <td className="px-6 py-4 text-right">
                      <div className="flex justify-end gap-2">
                        <button 
                          onClick={() => openModal(item)}
                          className="p-2 text-slate-400 hover:text-indigo-600 hover:bg-indigo-50 rounded-lg transition-all"
                          title="Editar"
                        >
                          <Edit3 size={18} />
                        </button>
                        <button 
                          onClick={() => deleteProduct(item.id)}
                          className="p-2 text-slate-400 hover:text-red-600 hover:bg-red-50 rounded-lg transition-all"
                          title="Eliminar"
                        >
                          <Trash2 size={18} />
                        </button>
                      </div>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
          {filteredInventory.length === 0 && (
            <div className="p-12 text-center text-slate-400">
              <Package size={48} className="mx-auto mb-4 opacity-20" />
              <p>No se encontraron productos en el inventario.</p>
            </div>
          )}
        </div>
      </div>

      {/* Modal Form */}
      {isModalOpen && (
        <div className="fixed inset-0 bg-slate-900/40 backdrop-blur-sm z-50 flex items-center justify-center p-4">
          <div className="bg-white rounded-2xl shadow-2xl w-full max-w-md overflow-hidden animate-in fade-in zoom-in duration-200">
            <div className="flex justify-between items-center p-6 border-b border-slate-100 bg-slate-50/50">
              <h2 className="text-xl font-bold text-slate-800">
                {editingProduct ? 'Editar Producto' : 'Añadir Producto'}
              </h2>
              <button onClick={closeModal} className="text-slate-400 hover:text-slate-600">
                <X size={24} />
              </button>
            </div>
            <form onSubmit={handleSaveProduct} className="p-6 space-y-4">
              <div>
                <label className="block text-sm font-semibold text-slate-700 mb-1">Nombre del Producto</label>
                <input 
                  name="name"
                  defaultValue={editingProduct?.name}
                  required
                  className="w-full px-4 py-2 rounded-lg border border-slate-200 focus:ring-2 focus:ring-indigo-500 outline-none"
                  placeholder="Ej: Teclado Mecánico"
                />
              </div>
              <div>
                <label className="block text-sm font-semibold text-slate-700 mb-1">Categoría</label>
                <input 
                  name="category"
                  defaultValue={editingProduct?.category}
                  required
                  className="w-full px-4 py-2 rounded-lg border border-slate-200 focus:ring-2 focus:ring-indigo-500 outline-none"
                  placeholder="Ej: Accesorios"
                />
              </div>
              <div className="grid grid-cols-2 gap-4">
                <div>
                  <label className="block text-sm font-semibold text-slate-700 mb-1">Precio ($)</label>
                  <input 
                    name="price"
                    type="number"
                    step="0.01"
                    defaultValue={editingProduct?.price}
                    required
                    className="w-full px-4 py-2 rounded-lg border border-slate-200 focus:ring-2 focus:ring-indigo-500 outline-none"
                  />
                </div>
                <div>
                  <label className="block text-sm font-semibold text-slate-700 mb-1">Stock Inicial</label>
                  <input 
                    name="stock"
                    type="number"
                    defaultValue={editingProduct?.stock}
                    required
                    className="w-full px-4 py-2 rounded-lg border border-slate-200 focus:ring-2 focus:ring-indigo-500 outline-none"
                  />
                </div>
              </div>
              <div>
                <label className="block text-sm font-semibold text-slate-700 mb-1">Stock Mínimo (Alerta)</label>
                <input 
                  name="minStock"
                  type="number"
                  defaultValue={editingProduct?.minStock || 5}
                  required
                  className="w-full px-4 py-2 rounded-lg border border-slate-200 focus:ring-2 focus:ring-indigo-500 outline-none"
                />
              </div>
              <div className="pt-4 flex gap-3">
                <button 
                  type="button"
                  onClick={closeModal}
                  className="flex-1 px-4 py-2 rounded-lg border border-slate-200 font-semibold text-slate-600 hover:bg-slate-50"
                >
                  Cancelar
                </button>
                <button 
                  type="submit"
                  className="flex-1 px-4 py-2 rounded-lg bg-indigo-600 text-white font-semibold hover:bg-indigo-700 shadow-lg shadow-indigo-100"
                >
                  {editingProduct ? 'Guardar Cambios' : 'Crear Producto'}
                </button>
              </div>
            </form>
          </div>
        </div>
      )}
    </div>
  );
};

const StatCard = ({ title, value, icon, color, alert = false }) => (
  <div className={`p-6 rounded-2xl border border-slate-200 bg-white shadow-sm flex items-center gap-4 transition-transform hover:-translate-y-1 ${alert ? 'ring-2 ring-amber-400' : ''}`}>
    <div className={`p-4 rounded-xl ${color}`}>
      {React.cloneElement(icon, { size: 28 })}
    </div>
    <div>
      <p className="text-sm font-medium text-slate-500 uppercase tracking-wider">{title}</p>
      <p className="text-2xl font-bold text-slate-800">{value}</p>
    </div>
  </div>
);

export default App;

```
