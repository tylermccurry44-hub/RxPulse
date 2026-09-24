# RxPulse
Medication management app
import React, { useState, useEffect } from 'react';
import { Bell, Plus, Camera, AlertCircle, DollarSign, CheckCircle, Trash2 } from 'lucide-react';

export default function MedManagerApp() {
  const [medications, setMedications] = useState(() => {
    const saved = localStorage.getItem('medications');
    return saved ? JSON.parse(saved) : [
      {
        id: 1,
        name: 'Lisinopril',
        dosage: '10mg',
        frequency: 'Once daily',
        doctor: 'Dr. Smith',
        purpose: 'Blood Pressure',
        supply: 7,
        totalRemaining: 5,
        refillAt: 10,
        photo: null,
        insuranceCovered: true,
        bestPrice: '$4.10 at CVS (with GoodRx)'
      }
    ];
  });

  const [showAddModal, setShowAddModal] = useState(false);
  const [form, setForm] = useState({
    name: '',
    dosage: '',
    frequency: 'Once daily',
    doctor: '',
    purpose: '',
    totalRemaining: 30,
    refillAt: 7,
    insuranceCovered: true
  });

  useEffect(() => {
    localStorage.setItem('medications', JSON.stringify(medications));
  }, [medications]);

  // Handle adding a new medication
  const handleSubmit = (e) => {
    e.preventDefault();
    const newMed = {
      id: Date.now(),
      ...form,
      bestPrice: 'Calculating local rates...'
    };
    setMedications([...medications, newMed]);
    setForm({ name: '', dosage: '', frequency: 'Once daily', doctor: '', purpose: '', totalRemaining: 30, refillAt: 7, insuranceCovered: true });
    setShowAddModal(false);
  };

  const deleteMed = (id) => {
    setMedications(medications.filter(med => med.id !== id));
  };

  // Handle mock prescription image upload (OCR simulation)
  const handleImageUpload = (e, id) => {
    const file = e.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onloadend = () => {
        setMedications(medications.map(med => med.id === id ? { ...med, photo: reader.result } : med));
      };
      reader.readAsDataURL(file);
    }
  };

  return (
    <div className="min-h-screen bg-slate-50 text-slate-800 p-4 md:p-8">
      <div className="max-w-4xl mx-auto">
        
        {/* Header */}
        <header className="flex justify-between items-center mb-8 bg-white p-6 rounded-2xl shadow-sm border border-slate-100">
          <div>
            <h1 className="text-2xl font-bold text-slate-900">RxPulse Manager</h1>
            <p className="text-sm text-slate-500">Track doses, refills, insurance plans, and local coupon pricing.</p>
          </div>
          <button 
            onClick={() => setShowAddModal(true)}
            className="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2.5 rounded-xl font-medium flex items-center gap-2 shadow-sm transition"
          >
            <Plus size={18} /> Add Prescription
          </button>
        </header>

        {/* Refill Alerts Banner */}
        {medications.some(m => m.totalRemaining <= m.refillAt) && (
          <div className="mb-6 bg-amber-50 border border-amber-200 p-4 rounded-2xl flex items-start gap-3">
            <AlertCircle className="text-amber-600 shrink-0 mt-0.5" size={20} />
            <div>
              <h2 className="font-semibold text-amber-900">Refill Action Required</h2>
              <p className="text-sm text-amber-700">One or more of your prescriptions are running low based on your daily usage.</p>
            </div>
          </div>
        )}

        {/* Medication Grid */}
        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
          {medications.map(med => (
            <div key={med.id} className="bg-white p-6 rounded-2xl shadow-sm border border-slate-100 flex flex-col justify-between">
              <div>
                <div className="flex justify-between items-start mb-3">
                  <div>
                    <span className="text-xs font-semibold uppercase tracking-wider text-blue-600 bg-blue-50 px-2.5 py-1 rounded-full">
                      {med.purpose}
                    </span>
                    <h3 className="text-xl font-bold text-slate-900 mt-2">{med.name} <span className="text-sm font-normal text-slate-500">({med.dosage})</span></h3>
                  </div>
                  <button onClick={() => deleteMed(med.id)} className="text-slate-400 hover:text-red-500 transition">
                    <Trash2 size={18} />
                  </button>
                </div>

                <div className="space-y-2 text-sm text-slate-600 my-4">
                  <p><strong>Frequency:</strong> {med.frequency}</p>
                  <p><strong>Prescribed by:</strong> {med.doctor}</p>
                  <p className={`font-medium ${med.totalRemaining <= med.refillAt ? 'text-red-600' : 'text-slate-700'}`}>
                    <strong>Supply Left:</strong> {med.totalRemaining} doses remaining (Refill alert at {med.refillAt})
                  </p>
                </div>

                {/* Pricing & Insurance Section */}
                <div className="bg-slate-50 p-3 rounded-xl border border-slate-100 mb-4">
                  <div className="flex items-center gap-2 text-xs font-semibold text-slate-700 mb-1">
                    <DollarSign size={14} className="text-emerald-600" />
                    <span>Best Local Price & Coupons</span>
                  </div>
                  <p className="text-xs text-slate-600">Insurance Status: {med.insuranceCovered ? <span className="text-emerald-600 font-medium">Covered</span> : <span className="text-amber-600 font-medium">Not Covered (Out-of-pocket)</span>}</p>
                  <p className="text-xs font-bold text-slate-800 mt-1">💡 {med.bestPrice}</p>
                </div>

                {/* Prescription Image Preview */}
                {med.photo && (
                  <div className="mb-4">
                    <img src={med.photo} alt="Prescription bottle" className="w-full h-32 object-cover rounded-xl border border-slate-200" />
                  </div>
                )}
              </div>

              <div className="flex items-center gap-3 pt-4 border-t border-slate-100">
                <label className="cursor-pointer bg-slate-100 hover:bg-slate-200 text-slate-700 px-3 py-2 rounded-xl text-xs font-medium flex items-center gap-1.5 transition">
                  <Camera size={14} /> Snap Rx Photo
                  <input type="file" accept="image/*" className="hidden" onChange={(e) => handleImageUpload(e, med.id)} />
                </label>
                <button 
                  onClick={() => setMedications(medications.map(m => m.id === med.id ? {...m, totalRemaining: Math.max(0, m.totalRemaining - 1)} : m))}
                  className="flex-1 bg-emerald-50 hover:bg-emerald-100 text-emerald-700 py-2 rounded-xl text-xs font-semibold transition text-center"
                >
                  Take Dose (-1)
                </button>
              </div>
            </div>
          ))}
        </div>

        {/* Modal for Adding Medication */}
        {showAddModal && (
          <div className="fixed inset-0 bg-slate-900/50 backdrop-blur-sm flex items-center justify-center p-4 z-50">
            <div className="bg-white rounded-3xl max-w-md w-full p-6 shadow-xl">
              <h2 className="text-xl font-bold text-slate-900 mb-4">Add New Prescription</h2>
              <form onSubmit={handleSubmit} className="space-y-4">
                <div>
                  <label className="block text-xs font-semibold uppercase text-slate-500 mb-1">Medication Name</label>
                  <input required type="text" value={form.name} onChange={e => setForm({...form, name: e.target.value})} className="w-full border border-slate-200 rounded-xl p-3 text-sm focus:outline-blue-600" placeholder="e.g. Atorvastatin" />
                </div>
                <div className="grid grid-cols-2 gap-3">
                  <div>
                    <label className="block text-xs font-semibold uppercase text-slate-500 mb-1">Dosage</label>
                    <input required type="text" value={form.dosage} onChange={e => setForm({...form, dosage: e.target.value})} className="w-full border border-slate-200 rounded-xl p-3 text-sm focus:outline-blue-600" placeholder="e.g. 20mg" />
                  </div>
                  <div>
                    <label className="block text-xs font-semibold uppercase text-slate-500 mb-1">Frequency</label>
                    <input required type="text" value={form.frequency} onChange={e => setForm({...form, frequency: e.target.value})} className="w-full border border-slate-200 rounded-xl p-3 text-sm focus:outline-blue-600" placeholder="e.g. Twice daily" />
                  </div>
                </div>
                <div className="grid grid-cols-2 gap-3">
                  <div>
                    <label className="block text-xs font-semibold uppercase text-slate-500 mb-1">Prescribing Doctor</label>
                    <input type="text" value={form.doctor} onChange={e => setForm({...form, doctor: e.target.value})} className="w-full border border-slate-200 rounded-xl p-3 text-sm focus:outline-blue-600" placeholder="e.g. Dr. Davis" />
                  </div>
                  <div>
                    <label className="block text-xs font-semibold uppercase text-slate-500 mb-1">What it's for</label>
                    <input type="text" value={form.purpose} onChange={e => setForm({...form, purpose: e.target.value})} className="w-full border border-slate-200 rounded-xl p-3 text-sm focus:outline-blue-600" placeholder="e.g. Cholesterol" />
                  </div>
                </div>
                <div className="flex justify-end gap-3 mt-6">
                  <button type="button" onClick={() => setShowAddModal(false)} className="px-4 py-2 text-sm font-medium text-slate-500 hover:bg-slate-100 rounded-xl">Cancel</button>
                  <button type="submit" className="px-5 py-2 bg-blue-600 text-white text-sm font-medium rounded-xl hover:bg-blue-700">Save Medication</button>
                </div>
              </form>
            </div>
          </div>
        )}

      </div>
    </div>
  );
}
