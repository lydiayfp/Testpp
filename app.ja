class InvestmentAnalytics {
    constructor() {
        this.symbols = {
            'global-etfs': ['SPY', 'QQQ', 'VUSA.L'],
            'unit-trusts': ['ABF.SI', 'NTA.SI'],
            'sg-stocks': ['O39.SI', 'D05.SI', 'U11.SI'],
            'us-stocks': ['AAPL', 'MSFT', 'GOOGL']
        };
        this.currentData = null;
        this.init();
    }

    init() {
        this.bindEvents();
        this.loadCachedData();
        this.refreshData();
        this.updateStatus('Ready');
    }

    bindEvents() {
        const selects = ['assetType', 'symbol', 'timeHorizon', 'metric'];
        selects.forEach(id => {
            document.getElementById(id).addEventListener('change', () => {
                this.renderChart();
                this.renderMetrics();
            });
        });

        const pullRefresh = document.getElementById('pullToRefresh');
        let startY = 0;
        
        pullRefresh.addEventListener('touchstart', (e) => {
            startY = e.touches[0].clientY;
        });
        
        pullRefresh.addEventListener('touchmove', (e) => {
            const currentY = e.touches[0].clientY;
            if (currentY - startY > 100) {
                pullRefresh.textContent = 'Refreshing...';
                pullRefresh.classList.add('refreshing');
                this.refreshData();
            }
        });

        pullRefresh.addEventListener('click', () => this.refreshData());

        // Auto-refresh on focus/visibility
        window.addEventListener('focus', () => this.refreshData());
        document.addEventListener('visibilitychange', () => {
            if (!document.hidden) this.refreshData();
        });
    }

    async refreshData() {
        this.updateStatus('Fetching live data...');
        try {
            // TODO: Replace with your market data API
            // Example: Alpha Vantage, Yahoo Finance API, or licensed exchange feed
            const response = await fetch('/api/quote?symbol=' + document.getElementById('symbol').value);
            const liveData = await response.json();
            
            // Update cache
            this.currentData = { ...this.currentData, ...liveData, lastUpdate: new Date().toISOString() };
            localStorage.setItem('investmentData', JSON.stringify(this.currentData));
            
            this.renderChart();
            this.renderMetrics();
            this.updateStatus(`Updated: ${new Date().toLocaleTimeString()}`);
        } catch (error) {
            console.warn('Live fetch failed, using cache:', error);
            this.updateStatus('Using cached data (offline)');
        }
    }

    loadCachedData() {
        const cached = localStorage.getItem('investmentData');
        if (cached) {
            this.currentData = JSON.parse(cached);
            this.renderChart();
            this.renderMetrics();
        } else {
            this.loadSampleData();
        }
    }

    loadSampleData() {
        // Sample data structure - replace with real API responses
        this.currentData = {
            symbol: 'SPY',
            exchange: 'NYSE',
            price: 528.45,
            ebitda: 45000000000,
            growth: 12.5,
            fcf: 42000000000,
            peg: 1.25,
            historical: this.generateSampleHistory()
        };
        localStorage.setItem('investmentData', JSON.stringify(this.currentData));
    }

    generateSampleHistory() {
        const now = new Date();
        const data = [];
        for (let i = 0; i < 260 * 20; i++) { // ~20 years trading days
            const date = new Date(now.getTime() - i * 24 * 60 * 60 * 1000 * 0.8);
            data.unshift({
                date: date.toISOString().split('T')[0],
                value: 400 + Math.sin(i / 50) * 100 + i * 0.08 + (Math.random() - 0.5) * 20
            });
        }
        return data;
    }

    renderChart() {
        if (!this.currentData?.historical) return;

        const timeHorizon = document.getElementById('timeHorizon').value;
        const metric = document.getElementById('metric').value;
        
        const years = parseInt(timeHorizon);
        const cutoffDate = new Date();
        cutoffDate.setFullYear(cutoffDate.getFullYear() - years);
        
        const filteredData = this.currentData.historical
            .filter(d => new Date(d.date) >= cutoffDate)
            .map(d => ({ x: d.date, y: d.value }));

        const trace = {
            x: filteredData.map(d => d.x),
            y: filteredData.map(d => d.y),
            type: 'scatter',
            mode: 'lines',
            line: { color: '#667eea', width: 3 },
            fill: 'tonexty'
        };

        Plotly.newPlot('chart', [trace], {
            title: `${this.currentData.symbol} - ${metric.toUpperCase()} (${timeHorizon})`,
            xaxis: { title: 'Date' },
            yaxis: { title: metric.toUpperCase() },
            showlegend: false,
            responsive: true
        });
    }

    renderMetrics() {
        if (!this.currentData) return;

        const metrics = {
            'share-price': { label: 'Current Price', value: `$${this.currentData.price?.toLocaleString()}` },
            'ebitda': { label: 'EBITDA', value: `$${this.formatNumber(this.currentData.ebitda)}` },
            'growth': { label: 'Growth %', value: `${this.currentData.growth}%` },
            'fcf': { label: 'Free Cash Flow', value: `$${this.formatNumber(this.currentData.fcf)}` },
            'peg': { label: 'PEG Ratio', value: this.currentData.peg.toFixed(2) }
        };

        const html = Object.entries(metrics).map(([key, data]) => `
            <div class="metric-card">
                <div class="label">${data.label}</div>
                <div class="value">${data.value}</div>
            </div>
        `).join('');

        document.getElementById('metrics-display').innerHTML = html;
    }

    formatNumber(num) {
        return num?.toLocaleString() || 'N/A';
    }

    updateStatus(message) {
        document.getElementById('status').textContent = message;
    }
}

// Initialize app
document.addEventListener('DOMContentLoaded', () => {
    new InvestmentAnalytics();
});
