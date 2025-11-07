<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fruit Finder: Nutrition & Natural Sugars</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Custom styles for the Inter font and mobile-first container */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f7fdee; /* Very light, fruity green */
        }
        .container-mobile {
            max-width: 100%;
            padding: 0.5rem;
        }
        @media (min-width: 640px) {
            .container-mobile {
                max-width: 640px; /* Max width for larger screens */
                margin: auto;
                padding: 1rem;
            }
        }
        .fruit-card {
            transition: transform 0.2s, box-shadow 0.2s;
        }
        .fruit-card:hover {
            transform: translateY(-3px);
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }
    </style>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        'primary-green': '#4ade80',
                        'accent-red': '#f87171',
                        'light-yellow': '#fde047',
                    }
                }
            }
        }
    </script>
</head>
<body class="min-h-screen">

    <div id="app" class="container-mobile pt-4 pb-16">
        <!-- Header -->
        <header class="text-center mb-6 p-4 bg-white rounded-xl shadow-lg">
            <h1 class="text-3xl font-extrabold text-primary-green tracking-tight">
                🍎 Fruit Finder 🥭
            </h1>
            <p class="text-sm text-gray-500 mt-1">Browse Nutrition & Natural Sugars (per 100g serving)</p>
        </header>

        <!-- Search Bar -->
        <div class="mb-6 sticky top-0 z-10 p-2 bg-f7fdee -mx-0.5">
            <input type="text" id="search-input" oninput="filterFruits()"
                   placeholder="Search for a fruit..."
                   class="w-full p-3 border-2 border-primary-green rounded-xl shadow-md focus:ring-2 focus:ring-primary-green focus:border-transparent transition duration-150">
        </div>

        <!-- Fruit Grid -->
        <div id="fruit-grid" class="grid grid-cols-2 sm:grid-cols-3 gap-3">
            <!-- Fruit cards will be injected here by JavaScript -->
        </div>

        <!-- Empty State/No Results Message -->
        <div id="no-results" class="text-center text-gray-500 mt-10 hidden">
            <p class="text-lg">No fruits matched your search. Try a different name!</p>
        </div>

        <!-- Detail Modal (Hidden by default) -->
        <div id="detail-modal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50 hidden" onclick="hideModal(event)">
            <div id="modal-content" class="bg-white p-6 rounded-2xl shadow-2xl w-full max-w-sm transform transition-all duration-300 scale-95 opacity-0" onclick="event.stopPropagation()">
                <h2 id="modal-fruit-name" class="text-3xl font-bold mb-4 text-center text-gray-800"></h2>
                <div class="text-5xl text-center mb-4" id="modal-fruit-emoji"></div>
                <div class="space-y-3">
                    <div class="flex justify-between items-center p-2 bg-yellow-50 rounded-lg">
                        <span class="font-semibold text-gray-700">☀️ Natural Sugar (g):</span>
                        <span id="modal-sugar" class="text-xl font-extrabold text-accent-red"></span>
                    </div>
                    <div class="flex justify-between items-center p-2 bg-green-50 rounded-lg">
                        <span class="font-semibold text-gray-700">🔥 Calories (kcal):</span>
                        <span id="modal-calories" class="text-xl font-extrabold text-primary-green"></span>
                    </div>
                    <div class="flex justify-between items-center p-2 bg-blue-50 rounded-lg">
                        <span class="font-semibold text-gray-700">💧 Water Content (%):</span>
                        <span id="modal-water" class="text-xl font-extrabold text-blue-500"></span>
                    </div>
                    <div class="flex justify-between items-center p-2 bg-indigo-50 rounded-lg">
                        <span class="font-semibold text-gray-700">🌾 Fiber (g):</span>
                        <span id="modal-fiber" class="text-xl font-extrabold text-indigo-500"></span>
                    </div>
                </div>
                <button onclick="hideModal()" class="mt-6 w-full py-2 bg-primary-green text-white font-bold rounded-xl hover:bg-green-600 transition duration-150 shadow-md">
                    Close
                </button>
            </div>
        </div>

    </div>

    <script>
        // --- 1. FRUIT DATASET (per 100g serving) ---
        const FRUITS_DATA = [
            { name: "Apple", emoji: "🍎", calories: 52, sugar: 10.4, water: 86, fiber: 2.4 },
            { name: "Banana", emoji: "🍌", calories: 89, sugar: 12.2, water: 75, fiber: 2.6 },
            { name: "Orange", emoji: "🍊", calories: 47, sugar: 9.4, water: 87, fiber: 2.4 },
            { name: "Strawberry", emoji: "🍓", calories: 32, sugar: 4.9, water: 91, fiber: 2.0 },
            { name: "Grape", emoji: "🍇", calories: 69, sugar: 16.0, water: 81, fiber: 0.9 },
            { name: "Mango", emoji: "🥭", calories: 60, sugar: 13.7, water: 83, fiber: 1.6 },
            { name: "Pineapple", emoji: "🍍", calories: 50, sugar: 9.9, water: 86, fiber: 1.4 },
            { name: "Avocado", emoji: "🥑", calories: 160, sugar: 0.7, water: 73, fiber: 6.7 },
            { name: "Kiwi", emoji: "🥝", calories: 61, sugar: 8.9, water: 83, fiber: 3.0 },
            { name: "Watermelon", emoji: "🍉", calories: 30, sugar: 6.2, water: 91, fiber: 0.4 },
            { name: "Cherry", emoji: "🍒", calories: 50, sugar: 8.0, water: 82, fiber: 2.1 },
            { name: "Lemon", emoji: "🍋", calories: 29, sugar: 2.5, water: 89, fiber: 2.8 },
            { name: "Peach", emoji: "🍑", calories: 39, sugar: 8.4, water: 89, fiber: 1.5 },
            { name: "Plum", emoji: "🫐", calories: 46, sugar: 9.9, water: 87, fiber: 1.4 },
            { name: "Coconut", emoji: "🥥", calories: 354, sugar: 6.2, water: 47, fiber: 9.0 },
        ];

        // --- 2. RENDERING FUNCTIONS ---

        const fruitGrid = document.getElementById('fruit-grid');
        const noResults = document.getElementById('no-results');
        const detailModal = document.getElementById('detail-modal');
        const modalContent = document.getElementById('modal-content');

        /**
         * Creates the HTML card for a single fruit.
         * @param {object} fruit - The fruit data object.
         * @returns {string} The HTML string for the fruit card.
         */
        function createFruitCard(fruit) {
            return `
                <div class="fruit-card bg-white p-3 rounded-xl shadow-md flex flex-col items-center justify-center cursor-pointer border border-gray-100"
                     onclick="showModal('${fruit.name}')"
                     role="button" tabindex="0">
                    <div class="text-4xl mb-1">${fruit.emoji}</div>
                    <h3 class="text-sm font-semibold text-center text-gray-800">${fruit.name}</h3>
                    <p class="text-xs text-gray-600 mt-1">
                        Sugar: <span class="font-bold text-accent-red">${fruit.sugar}g</span>
                    </p>
                </div>
            `;
        }

        /**
         * Renders the list of fruits based on the provided array.
         * @param {array} fruitsToDisplay - Array of fruit objects.
         */
        function renderFruits(fruitsToDisplay) {
            fruitGrid.innerHTML = '';
            if (fruitsToDisplay.length === 0) {
                noResults.classList.remove('hidden');
                return;
            }

            noResults.classList.add('hidden');
            const fragment = document.createDocumentFragment();

            fruitsToDisplay.forEach(fruit => {
                const tempDiv = document.createElement('div');
                tempDiv.innerHTML = createFruitCard(fruit).trim();
                fragment.appendChild(tempDiv.firstChild);
            });
            fruitGrid.appendChild(fragment);
        }

        // --- 3. INTERACTION/MODAL FUNCTIONS ---

        /**
         * Filters the fruit list based on the search input value.
         */
        function filterFruits() {
            const query = document.getElementById('search-input').value.toLowerCase().trim();
            const filtered = FRUITS_DATA.filter(fruit =>
                fruit.name.toLowerCase().includes(query)
            );
            renderFruits(filtered);
        }

        /**
         * Shows the detail modal for a specific fruit.
         * @param {string} fruitName - The name of the fruit to display.
         */
        function showModal(fruitName) {
            const fruit = FRUITS_DATA.find(f => f.name === fruitName);
            if (!fruit) return;

            // Populate Modal Content
            document.getElementById('modal-fruit-name').textContent = fruit.name;
            document.getElementById('modal-fruit-emoji').textContent = fruit.emoji;
            document.getElementById('modal-sugar').textContent = fruit.sugar.toFixed(1);
            document.getElementById('modal-calories').textContent = fruit.calories;
            document.getElementById('modal-water').textContent = fruit.water;
            document.getElementById('modal-fiber').textContent = fruit.fiber.toFixed(1);

            // Show Modal with Animation
            detailModal.classList.remove('hidden');
            setTimeout(() => {
                modalContent.classList.remove('opacity-0', 'scale-95');
                modalContent.classList.add('opacity-100', 'scale-100');
            }, 10);
        }

        /**
         * Hides the detail modal.
         * @param {Event} event - Optional event object for click-outside detection.
         */
        function hideModal(event) {
            // Hide only if clicking the backdrop or the close button
            if (event && event.target !== detailModal) return;

            // Animate out
            modalContent.classList.remove('opacity-100', 'scale-100');
            modalContent.classList.add('opacity-0', 'scale-95');

            // Hide the modal after transition
            setTimeout(() => {
                detailModal.classList.add('hidden');
            }, 300);
        }

        // --- 4. INITIALIZATION ---

        // Initial render of all fruits when the page loads
        document.addEventListener('DOMContentLoaded', () => {
            renderFruits(FRUITS_DATA);
        });

    </script>
</body>
</html>

