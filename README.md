import React, { useState, useEffect } from 'react';
import { ShoppingCart, Stethoscope, Home, Package, CalendarCheck, XCircle, Sparkles } from 'lucide-react';

// Mock Data for Medicines
const initialMedicines = [
  {
    id: 'med1',
    name: 'Paracetamol 500mg',
    description: 'Relieves pain and reduces fever.',
    price: 5.50,
    imageUrl: 'https://placehold.co/100x100/ADD8E6/000000?text=Paracetamol',
  },
  {
    id: 'med2',
    name: 'Amoxicillin 250mg',
    description: 'Antibiotic for bacterial infections.',
    price: 12.75,
    imageUrl: 'https://placehold.co/100x100/90EE90/000000?text=Amoxicillin',
  },
  {
    id: 'med3',
    name: 'Ibuprofen 400mg',
    description: 'Anti-inflammatory for pain and swelling.',
    price: 7.20,
    imageUrl: 'https://placehold.co/100x100/FFB6C1/000000?text=Ibuprofen',
  },
  {
    id: 'med4',
    name: 'Vitamin C 1000mg',
    description: 'Boosts immune system.',
    price: 8.99,
    imageUrl: 'https://placehold.co/100x100/FFD700/000000?text=Vitamin+C',
  },
  {
    id: 'med5',
    name: 'Cough Syrup',
    description: 'Relieves cough and cold symptoms.',
    price: 9.10,
    imageUrl: 'https://placehold.co/100x100/DDA0DD/000000?text=Cough+Syrup',
  },
];

// Mock Data for Doctors
const initialDoctors = [
  {
    id: 'doc1',
    name: 'Dr. Sarah Chen',
    specialty: 'General Practitioner',
    imageUrl: 'https://placehold.co/100x100/B0C4DE/000000?text=Dr.+Chen',
    availableTimes: ['9:00 AM', '10:00 AM', '2:00 PM', '3:00 PM'],
  },
  {
    id: 'doc2',
    name: 'Dr. Michael Lee',
    specialty: 'Pediatrician',
    imageUrl: 'https://placehold.co/100x100/FFDEAD/000000?text=Dr.+Lee',
    availableTimes: ['9:30 AM', '11:00 AM', '1:30 PM', '4:00 PM'],
  },
  {
    id: 'doc3',
    name: 'Dr. Emily White',
    specialty: 'Dermatologist',
    imageUrl: 'https://placehold.co/100x100/E0FFFF/000000?text=Dr.+White',
    availableTimes: ['10:00 AM', '11:30 AM', '3:00 PM', '4:30 PM'],
  },
  {
    id: 'doc4',
    name: 'Dr. David Kim',
    specialty: 'Cardiologist',
    imageUrl: 'https://placehold.co/100x100/F08080/000000?text=Dr.+Kim',
    availableTimes: ['8:30 AM', '10:30 AM', '1:00 PM', '3:30 PM'],
  },
];

// Main App Component
const App = () => {
  // State to manage the current page view
  const [currentPage, setCurrentPage] = useState('home');
  // State to manage items in the shopping cart
  const [cart, setCart] = useState([]);
  // State to manage the booked appointment details
  const [bookedAppointment, setBookedAppointment] = useState(null);
  // State to manage the selected doctor for booking
  const [selectedDoctor, setSelectedDoctor] = useState(null);
  // State to manage the selected time slot for booking
  const [selectedTime, setSelectedTime] = useState(null);
  // State to manage the visibility of the booking confirmation modal
  const [showBookingModal, setShowBookingModal] = useState(false);
  // State for search term in medicine list
  const [searchTerm, setSearchTerm] = useState('');

  // New states for AI features
  const [aiMedicineDescription, setAiMedicineDescription] = useState('');
  const [showMedicineAIModal, setShowMedicineAIModal] = useState(false);
  const [loadingMedicineAI, setLoadingMedicineAI] = useState(false);
  const [selectedMedicineForAI, setSelectedMedicineForAI] = useState(null);

  const [aiDoctorTips, setAiDoctorTips] = useState('');
  const [loadingDoctorAI, setLoadingDoctorAI] = useState(false);

  // Function to call Gemini API
  const callGeminiAPI = async (prompt) => {
    try {
      const chatHistory = [{ role: "user", parts: [{ text: prompt }] }];
      const payload = { contents: chatHistory };
      const apiKey = ""; // Canvas will provide this API key
      const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

      const response = await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      const result = await response.json();

      if (result.candidates && result.candidates.length > 0 &&
          result.candidates[0].content && result.candidates[0].content.parts &&
          result.candidates[0].content.parts.length > 0) {
        return result.candidates[0].content.parts[0].text;
      } else {
        console.error('Gemini API response structure unexpected:', result);
        return 'Could not generate information. Please try again.';
      }
    } catch (error) {
      console.error('Error calling Gemini API:', error);
      return 'Failed to connect to AI service. Please check your network.';
    }
  };

  // Function to get AI-powered medicine description
  const getAiMedicineDescription = async (medicine) => {
    setSelectedMedicineForAI(medicine);
    setLoadingMedicineAI(true);
    setAiMedicineDescription(''); // Clear previous description
    const prompt = `Provide a detailed, user-friendly explanation for ${medicine.name} (${medicine.description}). Include its primary uses, common side effects, and any important precautions a user should know. Keep it concise and easy to understand for a general audience.`;
    const response = await callGeminiAPI(prompt);
    setAiMedicineDescription(response);
    setLoadingMedicineAI(false);
    setShowMedicineAIModal(true);
  };

  // Function to get AI-powered doctor consultation tips
  const getAiDoctorTips = async (doctor) => {
    setLoadingDoctorAI(true);
    setAiDoctorTips(''); // Clear previous tips
    const prompt = `Generate 3-5 key questions or discussion points a patient should consider asking a ${doctor.specialty} like Dr. ${doctor.name.split(' ')[1]} during a consultation. Focus on general advice relevant to their specialty.`;
    const response = await callGeminiAPI(prompt);
    setAiDoctorTips(response);
    setLoadingDoctorAI(false);
  };

  // Function to add an item to the cart
  const addToCart = (medicine) => {
    setCart((prevCart) => {
      const existingItem = prevCart.find((item) => item.id === medicine.id);
      if (existingItem) {
        // If item already exists, increase quantity
        return prevCart.map((item) =>
          item.id === medicine.id ? { ...item, quantity: item.quantity + 1 } : item
        );
      } else {
        // Otherwise, add new item with quantity 1
        return [...prevCart, { ...medicine, quantity: 1 }];
      }
    });
  };

  // Function to remove an item from the cart
  const removeFromCart = (medicineId) => {
    setCart((prevCart) => prevCart.filter((item) => item.id !== medicineId));
  };

  // Function to increase quantity of an item in the cart
  const increaseQuantity = (medicineId) => {
    setCart((prevCart) =>
      prevCart.map((item) =>
        item.id === medicineId ? { ...item, quantity: item.quantity + 1 } : item
      )
    );
  };

  // Function to decrease quantity of an item in the cart
  const decreaseQuantity = (medicineId) => {
    setCart((prevCart) =>
      prevCart.map((item) =>
        item.id === medicineId && item.quantity > 1
          ? { ...item, quantity: item.quantity - 1 }
          : item
      ).filter(item => item.quantity > 0) // Remove if quantity becomes 0
    );
  };

  // Calculate total cost of items in the cart
  const getTotalCost = () => {
    return cart.reduce((total, item) => total + item.price * item.quantity, 0).toFixed(2);
  };

  // Handle "Checkout" action (simulated)
  const handleCheckout = () => {
    if (cart.length === 0) {
      // Using alert for simplicity as per user's prompt, but normally would use a custom modal.
      // IMPORTANT: Do NOT use alert() or confirm() in production code.
      alert('Your cart is empty!');
      return;
    }
    // IMPORTANT: Do NOT use alert() or confirm() in production code.
    alert('Checkout successful! (This is a simulation)');
    setCart([]); // Clear cart after checkout
    setCurrentPage('home'); // Go back to home or a confirmation page
  };

  // Handle doctor selection
  const handleSelectDoctor = (doctor) => {
    setSelectedDoctor(doctor);
    setSelectedTime(null); // Reset time when doctor changes
    setAiDoctorTips(''); // Clear previous AI tips when a new doctor is selected
  };

  // Handle time slot selection
  const handleSelectTime = (time) => {
    setSelectedTime(time);
  };

  // Handle "Book Appointment" action (simulated)
  const handleBookAppointment = () => {
    if (!selectedDoctor || !selectedTime) {
      // IMPORTANT: Do NOT use alert() or confirm() in production code.
      alert('Please select a doctor and an available time slot.');
      return;
    }
    setBookedAppointment({ doctor: selectedDoctor, time: selectedTime });
    setShowBookingModal(true); // Show confirmation modal
    setSelectedDoctor(null); // Clear selection after booking
    setSelectedTime(null);
    setAiDoctorTips(''); // Clear AI tips after booking
  };

  // Close booking confirmation modal
  const closeBookingModal = () => {
    setShowBookingModal(false);
    setCurrentPage('home'); // Navigate back to home after closing modal
  };

  // Filter medicines based on search term
  const filteredMedicines = initialMedicines.filter(medicine =>
    medicine.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
    medicine.description.toLowerCase().includes(searchTerm.toLowerCase())
  );

  // Component for the navigation bar
  const Navbar = () => (
    <nav className="bg-gradient-to-r from-blue-600 to-blue-800 p-4 shadow-lg rounded-b-xl">
      <div className="container mx-auto flex flex-wrap justify-between items-center">
        <h1 className="text-white text-3xl font-bold font-inter tracking-wide">MediCare</h1>
        <div className="flex space-x-4 mt-2 md:mt-0">
          <button
            onClick={() => setCurrentPage('home')}
            className="flex items-center text-white px-4 py-2 rounded-full hover:bg-blue-700 transition-colors duration-300 shadow-md"
          >
            <Home className="mr-2" size={20} /> Home
          </button>
          <button
            onClick={() => setCurrentPage('buyMedicine')}
            className="flex items-center text-white px-4 py-2 rounded-full hover:bg-blue-700 transition-colors duration-300 shadow-md"
          >
            <Package className="mr-2" size={20} /> Buy Medicine
          </button>
          <button
            onClick={() => setCurrentPage('bookDoctor')}
            className="flex items-center text-white px-4 py-2 rounded-full hover:bg-blue-700 transition-colors duration-300 shadow-md"
          >
            <Stethoscope className="mr-2" size={20} /> Book Doctor
          </button>
          <button
            onClick={() => setCurrentPage('cart')}
            className="relative flex items-center text-white px-4 py-2 rounded-full hover:bg-blue-700 transition-colors duration-300 shadow-md"
          >
            <ShoppingCart className="mr-2" size={20} /> Cart
            {cart.length > 0 && (
              <span className="absolute -top-1 -right-1 bg-red-500 text-white text-xs font-bold rounded-full h-5 w-5 flex items-center justify-center">
                {cart.length}
              </span>
            )}
          </button>
        </div>
      </div>
    </nav>
  );

  // Component for the Home Page
  const HomePage = () => (
    <div className="flex flex-col items-center justify-center min-h-[calc(100vh-80px)] p-4 bg-gray-50">
      <div className="bg-white p-8 rounded-2xl shadow-xl text-center max-w-2xl w-full transform transition-all duration-500 hover:scale-105">
        <h2 className="text-5xl font-extrabold text-blue-700 mb-6 font-inter leading-tight">
          Welcome to MediCare
        </h2>
        <p className="text-gray-700 text-lg mb-8">
          Your trusted platform for buying medicines and booking doctor appointments online.
          We connect you with quality healthcare services right at your fingertips.
        </p>
        <div className="flex flex-col sm:flex-row justify-center space-y-4 sm:space-y-0 sm:space-x-6">
          <button
            onClick={() => setCurrentPage('buyMedicine')}
            className="bg-green-500 text-white px-8 py-4 rounded-full text-xl font-semibold shadow-lg hover:bg-green-600 transition-all duration-300 transform hover:scale-105 flex items-center justify-center"
          >
            <Package className="mr-3" size={24} /> Shop Medicines
          </button>
          <button
            onClick={() => setCurrentPage('bookDoctor')}
            className="bg-purple-500 text-white px-8 py-4 rounded-full text-xl font-semibold shadow-lg hover:bg-purple-600 transition-all duration-300 transform hover:scale-105 flex items-center justify-center"
          >
            <Stethoscope className="mr-3" size={24} /> Book a Doctor
          </button>
        </div>
      </div>
    </div>
  );

  // Component for the Buy Medicine Page
  const BuyMedicinePage = () => (
    <div className="container mx-auto p-6 bg-gray-50 min-h-[calc(100vh-80px)]">
      <h2 className="text-4xl font-bold text-blue-700 mb-8 text-center font-inter">Our Medicines</h2>
      <div className="mb-6 flex justify-center">
        <input
          type="text"
          placeholder="Search medicines..."
          className="p-3 border border-gray-300 rounded-full shadow-sm w-full max-w-md focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all duration-300"
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
        />
      </div>
      <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-8">
        {filteredMedicines.map((medicine) => (
          <div
            key={medicine.id}
            className="bg-white rounded-2xl shadow-lg p-6 flex flex-col items-center text-center transform transition-all duration-300 hover:scale-105 hover:shadow-xl border border-gray-100"
          >
            <img
              src={medicine.imageUrl}
              alt={medicine.name}
              className="w-28 h-28 object-cover rounded-full mb-4 border-4 border-blue-100 shadow-md"
              onError={(e) => { e.target.onerror = null; e.target.src = `https://placehold.co/100x100/CCCCCC/FFFFFF?text=${medicine.name.split(' ')[0]}`; }}
            />
            <h3 className="text-xl font-semibold text-gray-800 mb-2 font-inter">{medicine.name}</h3>
            <p className="text-gray-600 text-sm mb-3 h-12 overflow-hidden">{medicine.description}</p>
            <p className="text-blue-600 text-2xl font-bold mb-4">${medicine.price.toFixed(2)}</p>
            <div className="flex flex-col space-y-3 w-full">
              <button
                onClick={() => addToCart(medicine)}
                className="bg-blue-500 text-white px-6 py-3 rounded-full font-semibold shadow-md hover:bg-blue-600 transition-colors duration-300 flex items-center justify-center"
              >
                <ShoppingCart size={20} className="mr-2" /> Add to Cart
              </button>
              <button
                onClick={() => getAiMedicineDescription(medicine)}
                className="bg-purple-500 text-white px-6 py-3 rounded-full font-semibold shadow-md hover:bg-purple-600 transition-colors duration-300 flex items-center justify-center"
                disabled={loadingMedicineAI}
              >
                {loadingMedicineAI ? (
                  <span className="animate-pulse">Loading...</span>
                ) : (
                  <>
                    <Sparkles size={20} className="mr-2" /> Learn More with AI
                  </>
                )}
              </button>
            </div>
          </div>
        ))}
      </div>
      {filteredMedicines.length === 0 && (
        <p className="text-center text-gray-500 text-lg mt-10">No medicines found matching your search.</p>
      )}

      {/* AI Medicine Description Modal */}
      {showMedicineAIModal && selectedMedicineForAI && (
        <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
          <div className="bg-white rounded-2xl shadow-2xl p-8 max-w-lg w-full text-center relative transform scale-100 transition-all duration-300 ease-out">
            <button
              onClick={() => setShowMedicineAIModal(false)}
              className="absolute top-4 right-4 text-gray-400 hover:text-gray-600 transition-colors"
            >
              <XCircle size={28} />
            </button>
            <h3 className="text-3xl font-bold text-blue-700 mb-4 font-inter">
              Insights for {selectedMedicineForAI.name}
            </h3>
            {loadingMedicineAI ? (
              <div className="text-center text-gray-600 text-lg">
                <Sparkles className="animate-bounce mx-auto mb-4 text-blue-400" size={32} />
                Generating AI insights...
              </div>
            ) : (
              <p className="text-gray-700 text-md text-left leading-relaxed whitespace-pre-wrap">{aiMedicineDescription}</p>
            )}
            <button
              onClick={() => setShowMedicineAIModal(false)}
              className="mt-6 bg-blue-500 text-white px-6 py-3 rounded-full font-semibold shadow-md hover:bg-blue-600 transition-colors duration-300"
            >
              Close
            </button>
          </div>
        </div>
      )}
    </div>
  );

  // Component for the Shopping Cart Page
  const CartPage = () => (
    <div className="container mx-auto p-6 bg-gray-50 min-h-[calc(100vh-80px)]">
      <h2 className="text-4xl font-bold text-blue-700 mb-8 text-center font-inter">Your Shopping Cart</h2>
      {cart.length === 0 ? (
        <div className="text-center text-gray-600 text-xl p-10 bg-white rounded-2xl shadow-lg">
          <ShoppingCart size={48} className="mx-auto mb-4 text-gray-400" />
          Your cart is empty. Start adding some medicines!
          <button
            onClick={() => setCurrentPage('buyMedicine')}
            className="mt-6 bg-blue-500 text-white px-6 py-3 rounded-full font-semibold shadow-md hover:bg-blue-600 transition-colors duration-300 flex items-center justify-center mx-auto w-fit"
          >
            <Package size={20} className="mr-2" /> Browse Medicines
          </button>
        </div>
      ) : (
        <div className="bg-white rounded-2xl shadow-lg p-6">
          <div className="divide-y divide-gray-200">
            {cart.map((item) => (
              <div key={item.id} className="flex items-center justify-between py-4">
                <div className="flex items-center space-x-4">
                  <img
                    src={item.imageUrl}
                    alt={item.name}
                    className="w-20 h-20 object-cover rounded-xl border border-gray-200"
                    onError={(e) => { e.target.onerror = null; e.target.src = `https://placehold.co/80x80/CCCCCC/FFFFFF?text=${item.name.split(' ')[0]}`; }}
                  />
                  <div>
                    <h3 className="text-lg font-semibold text-gray-800 font-inter">{item.name}</h3>
                    <p className="text-gray-600 text-sm">${item.price.toFixed(2)} each</p>
                  </div>
                </div>
                <div className="flex items-center space-x-4">
                  <div className="flex items-center border border-gray-300 rounded-full px-3 py-1">
                    <button
                      onClick={() => decreaseQuantity(item.id)}
                      className="text-gray-600 hover:text-gray-800 text-xl font-bold"
                    >
                      -
                    </button>
                    <span className="mx-3 text-lg font-medium">{item.quantity}</span>
                    <button
                      onClick={() => increaseQuantity(item.id)}
                      className="text-gray-600 hover:text-gray-800 text-xl font-bold"
                    >
                      +
                    </button>
                  </div>
                  <p className="text-lg font-bold text-blue-600 w-20 text-right">${(item.price * item.quantity).toFixed(2)}</p>
                  <button
                    onClick={() => removeFromCart(item.id)}
                    className="text-red-500 hover:text-red-700 transition-colors duration-200 p-2 rounded-full hover:bg-red-50"
                  >
                    <XCircle size={24} />
                  </button>
                </div>
              </div>
            ))}
          </div>
          <div className="flex justify-between items-center mt-8 pt-4 border-t-2 border-blue-100">
            <h3 className="text-2xl font-bold text-gray-800 font-inter">Total:</h3>
            <p className="text-3xl font-extrabold text-blue-700">${getTotalCost()}</p>
          </div>
          <div className="mt-8 text-center">
            <button
              onClick={handleCheckout}
              className="bg-green-500 text-white px-8 py-4 rounded-full text-xl font-semibold shadow-lg hover:bg-green-600 transition-all duration-300 transform hover:scale-105 flex items-center justify-center mx-auto w-fit"
            >
              <CalendarCheck size={24} className="mr-3" /> Proceed to Checkout
            </button>
          </div>
        </div>
      )}
    </div>
  );

  // Component for the Book Doctor Page
  const BookDoctorPage = () => (
    <div className="container mx-auto p-6 bg-gray-50 min-h-[calc(100vh-80px)]">
      <h2 className="text-4xl font-bold text-blue-700 mb-8 text-center font-inter">Book an Appointment</h2>
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
        {initialDoctors.map((doctor) => (
          <div
            key={doctor.id}
            className={`bg-white rounded-2xl shadow-lg p-6 flex flex-col items-center text-center transform transition-all duration-300 hover:scale-105 hover:shadow-xl border-2 ${
              selectedDoctor?.id === doctor.id ? 'border-blue-500 ring-4 ring-blue-200' : 'border-gray-100'
            }`}
          >
            <img
              src={doctor.imageUrl}
              alt={doctor.name}
              className="w-32 h-32 object-cover rounded-full mb-4 border-4 border-blue-100 shadow-md"
              onError={(e) => { e.target.onerror = null; e.target.src = `https://placehold.co/100x100/CCCCCC/FFFFFF?text=${doctor.name.split(' ')[1]}`; }}
            />
            <h3 className="text-2xl font-semibold text-gray-800 mb-1 font-inter">{doctor.name}</h3>
            <p className="text-blue-600 text-lg mb-4 font-medium">{doctor.specialty}</p>
            <button
              onClick={() => handleSelectDoctor(doctor)}
              className={`px-6 py-3 rounded-full font-semibold shadow-md transition-colors duration-300 w-full ${
                selectedDoctor?.id === doctor.id
                  ? 'bg-blue-600 text-white'
                  : 'bg-blue-100 text-blue-700 hover:bg-blue-200'
              }`}
            >
              {selectedDoctor?.id === doctor.id ? 'Selected' : 'Select Doctor'}
            </button>
          </div>
        ))}
      </div>

      {selectedDoctor && (
        <div className="mt-12 bg-white rounded-2xl shadow-lg p-8 border-2 border-blue-100">
          <h3 className="text-3xl font-bold text-blue-700 mb-6 text-center font-inter">
            Available Times for {selectedDoctor.name}
          </h3>
          <div className="flex flex-wrap justify-center gap-4 mb-8">
            {selectedDoctor.availableTimes.map((time) => (
              <button
                key={time}
                onClick={() => handleSelectTime(time)}
                className={`px-6 py-3 rounded-full text-lg font-medium border-2 transition-all duration-300 ${
                  selectedTime === time
                    ? 'bg-green-500 text-white border-green-500 shadow-lg'
                    : 'bg-gray-100 text-gray-700 border-gray-300 hover:bg-gray-200'
                }`}
              >
                {time}
              </button>
            ))}
          </div>
          <div className="flex flex-col sm:flex-row justify-center space-y-4 sm:space-y-0 sm:space-x-4 mt-6">
            <button
              onClick={handleBookAppointment}
              className="bg-purple-500 text-white px-8 py-4 rounded-full text-xl font-semibold shadow-lg hover:bg-purple-600 transition-all duration-300 transform hover:scale-105 flex items-center justify-center disabled:opacity-50 disabled:cursor-not-allowed"
              disabled={!selectedTime}
            >
              <CalendarCheck size={24} className="mr-3" /> Confirm Appointment
            </button>
            <button
              onClick={() => getAiDoctorTips(selectedDoctor)}
              className="bg-orange-500 text-white px-8 py-4 rounded-full text-xl font-semibold shadow-lg hover:bg-orange-600 transition-all duration-300 transform hover:scale-105 flex items-center justify-center"
              disabled={loadingDoctorAI}
            >
              {loadingDoctorAI ? (
                <span className="animate-pulse">Loading...</span>
              ) : (
                <>
                  <Sparkles size={24} className="mr-3" /> Get Consultation Tips
                </>
              )}
            </button>
          </div>

          {aiDoctorTips && (
            <div className="mt-8 p-6 bg-blue-50 rounded-xl border border-blue-200 shadow-inner">
              <h4 className="text-2xl font-bold text-blue-700 mb-4 text-center">AI Consultation Tips</h4>
              <p className="text-gray-700 text-md leading-relaxed whitespace-pre-wrap">{aiDoctorTips}</p>
            </div>
          )}
        </div>
      )}

      {/* Booking Confirmation Modal */}
      {showBookingModal && bookedAppointment && (
        <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
          <div className="bg-white rounded-2xl shadow-2xl p-8 max-w-md w-full text-center relative transform scale-100 transition-all duration-300 ease-out">
            <button
              onClick={closeBookingModal}
              className="absolute top-4 right-4 text-gray-400 hover:text-gray-600 transition-colors"
            >
              <XCircle size={28} />
            </button>
            <CalendarCheck size={64} className="text-green-500 mx-auto mb-6" />
            <h3 className="text-3xl font-bold text-green-700 mb-4 font-inter">Appointment Confirmed!</h3>
            <p className="text-gray-700 text-lg mb-2">
              You have successfully booked an appointment with:
            </p>
            <p className="text-xl font-semibold text-blue-600 mb-1">
              {bookedAppointment.doctor.name} ({bookedAppointment.doctor.specialty})
            </p>
            <p className="text-xl font-semibold text-blue-600 mb-6">
              at {bookedAppointment.time}
            </p>
            <button
              onClick={closeBookingModal}
              className="bg-blue-500 text-white px-6 py-3 rounded-full font-semibold shadow-md hover:bg-blue-600 transition-colors duration-300"
            >
              Got It!
            </button>
          </div>
        </div>
      )}
    </div>
  );

  // Render the appropriate page based on currentPage state
  const renderPage = () => {
    switch (currentPage) {
      case 'home':
        return <HomePage />;
      case 'buyMedicine':
        return <BuyMedicinePage />;
      case 'cart':
        return <CartPage />;
      case 'bookDoctor':
        return <BookDoctorPage />;
      default:
        return <HomePage />;
    }
  };

  return (
    <div className="min-h-screen bg-gray-100 font-sans antialiased">
      <Navbar />
      {renderPage()}
    </div>
  );
};

export default App;
