const BASE_URL = 'https://profalexbutler.github.io/mgis130-company/data/';

async function renderAnalyticsPage() {
  try {
    // 1. Fetch sales and menu data concurrently without caching
    const [salesResponse, menuResponse] = await Promise.all([
      fetch(`${BASE_URL}sales.json`, { cache: 'no-store' }),
      fetch(`${BASE_URL}menu.json`, { cache: 'no-store' })
    ]);

    const sales = await salesResponse.json();
    const menu = await menuResponse.json();

    // 2. Map menu items by id for O(1) lookups
    const menuMap = new Map(menu.map(item => [item.id, item]));

    // 3. Track summary statistics
    let totalUnitsThisWeek = 0;
    let bestSeller = { name: 'N/A', units: -1 };

    const listElement = document.querySelector('#list');
    listElement.innerHTML = ''; // Clear existing content

    // 4. Render cards
    sales.forEach(sale => {
      const menuItem = menuMap.get(sale.menuItemId) || {
        name: 'Unknown Item',
        category: 'Uncategorized',
        price: 0
      };

      totalUnitsThisWeek += sale.unitsThisWeek;

      // Track the top seller of the week
      if (sale.unitsThisWeek > bestSeller.units) {
        bestSeller = {
          name: menuItem.name,
          units: sale.unitsThisWeek
        };
      }

      // Check alert condition: performance drop week-over-week
      const isSalesDrop = sale.unitsThisWeek < sale.unitsLastWeek;

      // Build card element
      const card = document.createElement('div');
      card.className = `card ${isSalesDrop ? 'alert' : ''}`;
      
      card.innerHTML = `
        <span class="tag">${menuItem.category}</span>
        <h3>${menuItem.name}</h3>
        <p class="price">$${menuItem.price.toFixed(2)}</p>
        <div class="metrics">
          <p><strong>This Week:</strong> ${sale.unitsThisWeek} units</p>
          <p><strong>Last Week:</strong> ${sale.unitsLastWeek} units</p>
        </div>
      `;

      listElement.appendChild(card);
    });

    // 5. Update Page Title
    document.querySelector('#title').textContent = 'Our Food Truck - Analytics (BI)';

    // 6. Update Summary Metrics
    document.querySelector('#summary').innerHTML = `
      <div class="summary-metric">
        <span class="label">Units Sold This Week</span>
        <span class="value">${totalUnitsThisWeek.toLocaleString()}</span>
      </div>
      <div class="summary-metric">
        <span class="label">This Week's Best Seller</span>
        <span class="value">${bestSeller.name} (${bestSeller.units} units)</span>
      </div>
    `;

  } catch (error) {
    console.error('Failed to load Analytics data:', error);
  }
}

// Initialize on page load
document.addEventListener('DOMContentLoaded', renderAnalyticsPage);
Clean CSS Stylesheet for Analytics Hooks
To ensure your cards and alerts render cleanly according to the contract structure, apply these CSS rules:

CSS
/* Page Setup */
#title {
  font-size: 1.8rem;
  margin-bottom: 1rem;
}

#summary {
  display: flex;
  gap: 1.5rem;
  margin-bottom: 2rem;
  background-color: #f4f4f6;
  padding: 1rem;
  border-radius: 8px;
}

.summary-metric {
  display: flex;
  flex-direction: column;
}

.summary-metric .label {
  font-size: 0.85rem;
  color: #666;
}

.summary-metric .value {
  font-size: 1.4rem;
  font-weight: bold;
}

/* Card Grid Layout */
#list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 1rem;
}

.card {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 1rem;
  position: relative;
  background-color: #ffffff;
}

/* Category Tag */
.tag {
  display: inline-block;
  background-color: #e2e8f0;
  color: #334155;
  font-size: 0.75rem;
  font-weight: bold;
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
  margin-bottom: 0.5rem;
}

/* Alert State: Week-Over-Week Drop */
.card.alert {
  border: 2px solid #ef4444;
  background-color: #fef2f2;
}

.card.alert::after {
  content: "⚠️ Sales Dropped";
  display: block;
  font-size: 0.75rem;
  color: #dc2626;
  font-weight: bold;
  margin-top: 0.5rem;
}
