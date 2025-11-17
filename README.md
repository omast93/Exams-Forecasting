<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dynamic Forecasting Tool with Factor Adjustments (V24 - Zero-Base Fix)</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Load Chart.js for graphs -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js@3.7.1/dist/chart.min.js"></script>
    <style>
        /* Styling for Inter font and aesthetics */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f4f7f9;
        }
        .dropzone {
            border: 2px dashed #9ca3af;
            background-color: #ffffff;
            transition: all 0.2s;
        }
        .dropzone.hover {
            border-color: #10B981;
            background-color: #ecfdf5;
        }
        .model-selector {
            transition: all 0.2s;
            cursor: pointer;
        }
        .model-selector.active {
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.06);
            transform: translateY(-2px);
            border-width: 3px;
        }
        .factor-label:hover {
            opacity: 0.85;
        }
    </style>
</head>
<body class="p-4 md:p-8">

    <div class="max-w-7xl mx-auto">
        <h1 class="text-3xl font-extrabold text-gray-800 mb-2">
            📊 Dynamic Forecast Control Panel (V24)
        </h1>
        <p class="text-gray-500 mb-6">Define your **Historical Base** for Year-over-Year (YoY) calculation, apply **External/Internal Factors**, and set the **Forecast Duration**.</p>
        
        <!-- File Upload Container -->
        <div id="file-upload-container" class="mb-8">
            <label for="csv-file" class="block text-sm font-medium text-gray-700 mb-2">Step 1: Upload CSV Data File</label>
            
            <div id="dropzone" class="dropzone p-6 text-center rounded-xl cursor-pointer hover:shadow-lg">
                <input type="file" id="csv-file" accept=".csv" class="hidden">
                <p class="text-gray-600 font-semibold">Drag and drop your CSV file here, or click to select it.</p>
                <p class="text-xs text-gray-400 mt-1">Required columns: Month, Year, and the metric to analyze (e.g., Exams).</p>
            </div>
            
            <div id="file-status" class="mt-3 text-sm font-medium text-gray-600 hidden">File Loaded: <span id="file-name" class="font-bold"></span></div>
        </div>

        <!-- Dynamic Controls (Hidden until file is loaded) -->
        <div id="controls-container" class="hidden">
            
            <!-- Step 2: Metric, Duration, and Model -->
            <label class="block text-lg font-bold text-gray-800 mb-3 mt-8">Step 2: Define Metric and Forecast Parameters</label>
            
            <div class="grid grid-cols-1 md:grid-cols-4 gap-4 mb-4 p-4 bg-white rounded-xl shadow">
                
                <div>
                    <label for="data-metric-selector" class="block text-sm font-medium text-gray-700 font-bold">Primary Metric to Project (e.g., Exams)</label>
                    <select id="data-metric-selector" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 h-10"></select>
                </div>

                <div>
                    <label for="forecast-duration-selector" class="block text-sm font-medium text-gray-700 font-bold">Forecast Duration</label>
                    <select id="forecast-duration-selector" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 h-10">
                        <option value="12">1 Year (12 Months)</option>
                        <option value="6" selected>6 Months</option>
                        <option value="3">3 Months</option>
                    </select>
                </div>
                
                <div>
                    <label for="budget-metric-selector" class="block text-sm font-medium text-gray-700">Budget/Target Line (Comparative)</label>
                    <select id="budget-metric-selector" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 h-10"></select>
                </div>

                <div class="md:col-span-1">
                    <label class="block text-sm font-medium text-gray-700 font-bold">Forecast Model</label>
                    <div class="model-selector mt-1 p-2 text-center border-2 border-indigo-600 bg-indigo-50 text-indigo-700 rounded-md active" data-model="yoy">
                        <p class="font-semibold text-base">Year-over-Year (YoY %)</p>
                    </div>
                </div>
            </div>

            <!-- Step 3: Historical Base Period Selection -->
            <label class="block text-lg font-bold text-gray-800 mb-3 mt-8">Step 3: Define Historical Base for YoY Calculation</label>
            <div class="grid grid-cols-1 md:grid-cols-4 gap-4 mb-6 p-4 bg-yellow-50 rounded-xl border border-yellow-200 shadow">
                
                <div class="md:col-span-1">
                    <label class="block text-sm font-medium text-yellow-700 font-bold">Historical Base Period:</label>
                    <p class="text-xs text-yellow-600">YoY growth is averaged ONLY within this range.</p>
                </div>

                <div>
                    <label for="forecast-start-selector" class="block text-sm font-medium text-gray-700">Start Date</label>
                    <select id="forecast-start-selector" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-yellow-500 focus:ring-yellow-500 h-10"></select>
                </div>

                <div>
                    <label for="forecast-basis-end-selector" class="block text-sm font-medium text-gray-700">End Date</label>
                    <select id="forecast-basis-end-selector" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-yellow-500 focus:ring-yellow-500 h-10"></select>
                </div>
                
                <div class="flex items-center justify-center">
                    <span id="calculated-yoy" class="text-2xl font-extrabold text-yellow-800 p-2 bg-yellow-200 rounded-lg">--% Average YoY</span>
                </div>
                
            </div>

            <!-- Step 4: External and Internal Factors -->
            <label class="block text-lg font-bold text-gray-800 mb-3 mt-8">Step 4: Select External and Internal Adjustment Factors</label>
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
                
                <!-- External Factors -->
                <div class="p-5 bg-red-50 rounded-xl border border-red-200 shadow-lg">
                    <h3 class="text-xl font-bold text-red-800 mb-3 border-b pb-2">External Factors (Applies to the entire Forecast)</h3>
                    <div id="external-factors-list" class="space-y-2">
                        <!-- Factors generated by JS based on FACTORS_CONFIG -->
                    </div>
                    <p class="text-xs text-red-600 mt-3 italic">These factors multiply the base forecast.</p>
                </div>
                
                <!-- Internal Factors -->
                <div class="p-5 bg-green-50 rounded-xl border border-green-200 shadow-lg">
                    <h3 class="text-xl font-bold text-green-800 mb-3 border-b pb-2">Internal Factors (Applies to the Target)</h3>
                    <div id="internal-factors-list" class="space-y-2">
                        <!-- Factors generated by JS based on FACTORS_CONFIG -->
                    </div>
                    <p class="text-xs text-green-600 mt-3 italic">These factors are additive to the external ones. **(The 6-month factor is a 15% reduction only for the first half-year)**</p>
                </div>
            </div>

            <!-- Step 5: Summary and Export -->
            <label class="block text-lg font-bold text-gray-800 mb-3 mt-8">Step 5: Forecast Summary</label>
            <div class="flex flex-col sm:flex-row justify-between items-stretch space-y-4 sm:space-y-0 sm:space-x-4 mb-8">
                
                <!-- Summary Card 1: Base Forecast -->
                <div class="bg-blue-100 p-4 rounded-lg w-full text-blue-800 border-l-4 border-blue-500 shadow-md">
                    <p class="text-base font-extrabold mb-1">Total Base Forecast (Unadjusted)</p>
                    <p class="font-bold text-2xl"><span id="summary-base-total" class="text-blue-700">--</span></p>
                    <p class="text-sm mt-1 text-blue-600">Calculated using the Average YoY from the base period.</p>
                </div>
                
                <!-- Summary Card 2: Adjusted Forecast -->
                <div class="bg-indigo-100 p-4 rounded-lg w-full text-indigo-800 border-l-4 border-indigo-500 shadow-md">
                    <p class="text-base font-extrabold mb-1">Total Adjusted Forecast (Final)</p>
                    <p class="font-bold text-2xl"><span id="summary-adjusted-total" class="text-indigo-700">--</span></p>
                    <p class="text-sm mt-1 text-indigo-600">Includes all selected External and Internal Factor Adjustments.</p>
                </div>
                
                <!-- Export Button -->
                <button id="export-chart-btn" class="bg-emerald-600 text-white font-semibold py-3 px-8 rounded-lg shadow-lg hover:bg-emerald-700 transition duration-300 flex items-center w-full sm:w-auto justify-center">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2" viewBox="0 0 20 20" fill="currentColor">
                        <path fill-rule="evenodd" d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm3.293-7.707a1 1 0 011.414 0L10 11.586l1.293-1.293a1 1 0 111.414 1.414l-2 2a1 1 0 01-1.414 0l-2-2a1 1 0 010-1.414z" clip-rule="evenodd" />
                        <path fill-rule="evenodd" d="M10 2a1 1 0 011 1v9a1 1 0 11-2 0V3a1 1 0 011-1z" clip-rule="evenodd" />
                    </svg>
                    Export Chart (.png)
                </button>
            </div>
        </div>

        <!-- Chart Container -->
        <div id="chart-container" class="bg-white p-6 rounded-xl shadow-2xl hidden">
            <h2 id="chart-title" class="text-xl font-bold text-gray-700 mb-4">Monthly Trend</h2>
            <canvas id="mainChart"></canvas>
            <p id="forecast-note" class="text-sm text-gray-500 mt-4 italic"></p>
        </div>
        
        <!-- Error and Alert Messages -->
        <div id="message-box" class="fixed bottom-4 right-4 p-4 rounded-lg shadow-xl text-white font-medium transition-opacity duration-300 hidden z-50"></div>

    </div>

    <script>
        // --- Global Variables and Constants ---
        let mainChart = null;
        let rawData = [];
        let numericHeaders = [];
        let currentForecastModel = 'yoy'; 
        
        // Dynamic User Selections
        let DATA_KEY = ''; 
        let BUDGET_KEY = ''; 
        let FORECAST_START_MONTH = '';      // Start month for YoY calculation base
        let FORECAST_START_YEAR = 0;        // Start year for YoY calculation base
        let BASIS_END_MONTH = '';           // End month for YoY calculation base
        let BASIS_END_YEAR = 0;             // End year for YoY calculation base
        let FORECAST_DURATION_MONTHS = 6;   // 3, 6, or 12

        const YOY_KEY = 'YoY %'; 
        const monthOrder = ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'];
        
        // --- Factor Configuration (Maps to Checkboxes) ---
        const FACTORS_CONFIG = {
            external: [
                { id: 'economic_downturn', label: 'Economic Downturn', factor: -0.20 }, // -20%
                { id: 'pandemic_restrictions', label: 'Pandemic/Restrictions', factor: -0.30 }, // -30%
                { id: 'country_sanctions', label: 'Country Sanctions', factor: -0.10 } // -10%
            ],
            internal: [
                { id: 'aggressive_marketing', label: 'Aggressive Marketing Push', factor: 0.08 }, // +8%
                // CORRECTION: Applied -15% (downturn) only for the first 6 months
                { id: 'new_product_launch', label: 'New Product Launch (6 Months)', factor: -0.15, duration: 6 }, // -15% for 6 months
                { id: 'team_restructure', label: 'Team Restructure/Training', factor: -0.05 } // -5%
            ]
        };

        // --- Utility Functions ---

        /** Displays alert messages. */
        function showMessage(message, type = 'info') {
            const box = document.getElementById('message-box');
            box.innerText = message;
            box.classList.remove('hidden', 'bg-red-500', 'bg-blue-500', 'bg-green-500', 'opacity-0');
            box.classList.add('opacity-100');
            
            if (type === 'error') {
                box.classList.add('bg-red-500');
            } else if (type === 'success') {
                box.classList.add('bg-green-500');
            } else {
                box.classList.add('bg-blue-500');
            }
            
            setTimeout(() => {
                box.classList.remove('opacity-100');
                box.classList.add('opacity-0');
                setTimeout(() => box.classList.add('hidden'), 300);
            }, 5000);
        }

        /** Formats a value (with currency if applicable). */
        function formatValue(value, key) {
            if (value === null || isNaN(value)) return '--';
            const isCurrency = key && (key.toLowerCase().includes('revenue') || key.toLowerCase().includes('sale') || key.toLowerCase().includes('budget') || key.toLowerCase().includes('price'));
            
            const options = { minimumFractionDigits: 0, maximumFractionDigits: 0 };
            
            if (isCurrency) {
                 // Use a generic currency symbol for examples
                 return '£' + Math.round(value).toLocaleString('en-US', options);
            } else {
                 return Math.round(value).toLocaleString('en-US', options);
            }
        }
        
        /** Cleans and converts a string to a numeric value. */
        function parseNumericValue(valueStr) {
            if (!valueStr) return NaN;
            let cleanStr = String(valueStr).trim().replace(/£|€|\$|,/g, ''); 
            
            // Handle comma as decimal separator if necessary, but generally assume US format
            if (cleanStr.includes(',')) {
                cleanStr = cleanStr.replace(/(\d+)\.(\d+)/g, '$1$2').replace(',', '.');
            }
            
            const value = parseFloat(cleanStr);
            return isNaN(value) ? NaN : value;
        }
        
        /** Cleans and converts a percentage string (e.g., "99%", "-25%") to a decimal. */
        function parseYoY(yoyStr) {
            if (!yoyStr) return NaN;
            let cleanStr = String(yoyStr).replace(/%/g, '').trim();
            const value = parseFloat(cleanStr);
            return isNaN(value) ? NaN : value / 100;
        }
        
        /** Helps to exclude non-numeric/metadata columns during detection. */
        function isDataMetric(header) {
            const excluded = ['Month', 'Year', YOY_KEY, 'Zero', 'Working Days:', 'Sale_PY_MTD', 'Sale_#_PY_MTD', 'Budget', 'Timeseries'];
            return !excluded.some(ex => header.toLowerCase().includes(ex.toLowerCase()));
        }

        /** Analyzes the CSV, cleans data, and identifies numeric headers. */
        function parseCSV(csv) {
            try {
                const lines = csv.trim().split(/\r?\n/).filter(line => line.trim() !== '');
                if (lines.length <= 1) {
                    showMessage("Error: CSV file is empty or only contains headers.", 'error');
                    return null;
                }

                const headerLine = lines[0];
                const headers = headerLine.split(',').map(h => h.trim().replace(/"/g, ''));
                
                if (!headers.includes('Month') || !headers.includes('Year')) {
                    showMessage("Error: CSV must contain the required columns: 'Month' and 'Year'.", 'error');
                    return null;
                }

                const data = [];
                const potentialNumericHeaders = [];

                // Detection of numeric headers
                const detectionRows = lines.slice(1, Math.min(10, lines.length)); 

                headers.forEach(header => {
                    if (isDataMetric(header)) {
                        const isNumeric = detectionRows.some(line => {
                            const parts = line.split(/,(?=(?:(?:[^"]*"){2})*[^"]*$)/); 
                            const index = headers.indexOf(header);
                            if (index !== -1) {
                                const valueStr = (parts[index] || '').replace(/^"|"$/g, '').trim();
                                return !isNaN(parseNumericValue(valueStr));
                            }
                            return false;
                        });
                        if (isNumeric) {
                            potentialNumericHeaders.push(header);
                        }
                    }
                });
                
                numericHeaders = potentialNumericHeaders;

                for (let i = 1; i < lines.length; i++) {
                    const currentLine = lines[i];
                    // Split by comma, respecting quotes
                    const parts = currentLine.split(/,(?=(?:(?:[^"]*"){2})*[^"]*$)/); 
                    
                    const cleanRow = {};
                    
                    headers.forEach((header, index) => {
                        const valueStr = (parts[index] || '').replace(/^"|"$/g, '').trim();
                        cleanRow[header] = valueStr;
                    });
                    
                    cleanRow.Year = parseInt(cleanRow.Year);
                    cleanRow[YOY_KEY] = parseYoY(cleanRow[YOY_KEY] || 0);

                    numericHeaders.forEach(header => {
                        cleanRow[header] = parseNumericValue(cleanRow[header]);
                    });

                    // Parse other common numeric-like columns
                    const budgetHeader = headers.find(h => h.toLowerCase().includes('budget'));
                    if (budgetHeader) {
                        cleanRow[budgetHeader] = parseNumericValue(cleanRow[budgetHeader]);
                    }
                    
                    if (!isNaN(cleanRow.Year) && monthOrder.includes(cleanRow.Month)) {
                        data.push(cleanRow);
                    }
                }
                
                if (data.length === 0 || numericHeaders.length === 0) {
                    showMessage("Error: No valid time series data found. Check 'Month', 'Year', and ensure there is at least one numeric column.", 'error');
                    return null;
                }
                
                return data;
            } catch (e) {
                console.error("Error parsing CSV:", e);
                showMessage("Critical error processing the CSV file.", 'error');
                return null;
            }
        }
        
        // --- Forecasting Logic ---

        /** * Calculates the average YoY growth from a defined historical range, 
         * and the average absolute historical value (for zero-base fallback).
         */
        function calculateAverageYoY(data, primaryKey, startMonth, startYear, endMonth, endYear) {
            
            const startPeriod = startYear * 12 + monthOrder.indexOf(startMonth);
            const endPeriod = endYear * 12 + monthOrder.indexOf(endMonth);
            
            let yoyValues = [];
            let historicalValues = []; // Added for absolute average
            
            // Iterate over data and filter by the range
            data.forEach(d => {
                const currentPeriod = d.Year * 12 + monthOrder.indexOf(d.Month);
                
                // 1. Must be within the range (inclusive)
                if (currentPeriod >= startPeriod && currentPeriod <= endPeriod) {
                    
                    // 2. Must have data for the primary metric and YoY
                    if (!isNaN(d[primaryKey]) && !isNaN(d[YOY_KEY])) { 
                         // Use the provided YoY percentage for that period
                         yoyValues.push(d[YOY_KEY]);
                    }
                    
                    // Collect all valid historical values in the range
                    if (!isNaN(d[primaryKey])) {
                        historicalValues.push(d[primaryKey]);
                    }
                }
            });
            
            // --- Calculate Avg YoY (Percentage) ---
            let avgYoY = 0;
            if (yoyValues.length > 0) {
                 const sumYoY = yoyValues.reduce((sum, val) => sum + val, 0);
                 avgYoY = sumYoY / yoyValues.length;
            } else {
                 // Do not show warning here, handled by caller
            }

            // --- Calculate Avg Historical Value (Absolute) ---
            const sumHistorical = historicalValues.reduce((sum, val) => sum + val, 0);
            const avgHistoricalValue = historicalValues.length > 0 ? sumHistorical / historicalValues.length : 0;
            
            return { avgYoY, avgHistoricalValue };
        }

        /** Gets the last historical point with data for the Primary Metric. */
        function getLastHistoricalPoint(data, primaryKey) {
            const historicalMonths = data
                .filter(d => !isNaN(d[primaryKey]) && monthOrder.includes(d.Month))
                .sort((a, b) => (a.Year * 12 + monthOrder.indexOf(a.Month)) - (b.Year * 12 + monthOrder.indexOf(b.Month)));
                
            return historicalMonths.length > 0 ? historicalMonths[historicalMonths.length - 1] : null;
        }

        /** Generates all points (Historical + Forecast) based on selections and factors. */
        function generateForecastData(data, primaryKey, budgetKey, 
                                      startBasisMonth, startBasisYear, endBasisMonth, endBasisYear,
                                      durationMonths) {
                                          
            const sortedData = data.sort((a, b) => (a.Year * 12 + monthOrder.indexOf(a.Month)) - (b.Year * 12 + monthOrder.indexOf(b.Month)));
            
            const lastHistoricalPoint = getLastHistoricalPoint(sortedData, primaryKey);
            if (!lastHistoricalPoint) return { annualForecastData: [], totalBase: 0, totalAdjusted: 0 };

            const lastHistoricalPeriodIndex = lastHistoricalPoint.Year * 12 + monthOrder.indexOf(lastHistoricalPoint.Month);
            const startForecastPeriodIndex = lastHistoricalPeriodIndex + 1;
            const targetPeriodEndIndex = lastHistoricalPeriodIndex + durationMonths;
            
            // --- 1. Mapping Historical Points ---
            const historicalMap = new Map(); 
            sortedData.forEach(d => {
                historicalMap.set(`${d.Month}${d.Year}`, d);
            });
            
            // --- 2. Calculate Average YoY Growth (BASE) ---
            const { avgYoY, avgHistoricalValue } = calculateAverageYoY(
                sortedData, primaryKey, 
                startBasisMonth, startBasisYear, 
                endBasisMonth, endBasisYear
            );

            const avgYoYMultiplier = 1 + avgYoY;
            const avgYoYLabel = (avgYoY * 100).toFixed(1) + '%';
            
            // --- 3. Get Selected Factors ---
            const selectedExternalFactors = Array.from(document.querySelectorAll('#external-factors-list input:checked')).map(cb => {
                return FACTORS_CONFIG.external.find(f => f.id === cb.value);
            });
            const selectedInternalFactors = Array.from(document.querySelectorAll('#internal-factors-list input:checked')).map(cb => {
                return FACTORS_CONFIG.internal.find(f => f.id === cb.value);
            });

            const annualForecastData = [];
            let totalBaseForecast = 0;
            let totalAdjustedForecast = 0;
            
            const startYear = sortedData[0].Year;

            // Iterate through all periods from the start of data to the end of the forecast
            for (let currentPeriod = startYear * 12; currentPeriod <= targetPeriodEndIndex; currentPeriod++) {
                
                const year = Math.floor(currentPeriod / 12);
                const monthIndex = currentPeriod % 12;
                const month = monthOrder[monthIndex];
                
                const monthYearKey = `${month}${year}`;
                const historicalPoint = historicalMap.get(monthYearKey);
                
                let primaryValue = null;
                let budgetValue = null;
                let isForecast = false;
                let forecastBasis = 'N/A';
                
                const isFutureMonth = (currentPeriod >= startForecastPeriodIndex);
                
                const rawPrimaryValue = historicalPoint ? historicalPoint[primaryKey] : NaN;
                
                if (historicalPoint && !isNaN(rawPrimaryValue) && !isFutureMonth) {
                    // --- Historical Data (Actual) ---
                    primaryValue = rawPrimaryValue;
                    budgetValue = historicalPoint[budgetKey]; 

                } else if (isFutureMonth) {
                    
                    // --- Forecast Data (Projected) ---
                    isForecast = true;
                    
                    // 4. Calculate Base Forecast Value (YoY from Previous Year)
                    const previousYearPoint = historicalMap.get(`${month}${year - 1}`);
                    const baseValueYoY = previousYearPoint ? previousYearPoint[primaryKey] : NaN;
                    
                    let baseForecastValue;
                    
                    // FIX FOR ZERO/MISSING BASE: If the previous year's value is 0 or missing (NaN), use the overall average historical value as the base.
                    if (isNaN(baseValueYoY) || baseValueYoY === 0) {
                        baseForecastValue = avgHistoricalValue * avgYoYMultiplier;
                        // Provide detailed basis for the user
                        forecastBasis = `YoY Base (Avg. Historical: ${formatValue(avgHistoricalValue, primaryKey)} * ${avgYoYLabel})`;
                    } else {
                        // Standard calculation: PY value * (1 + Avg YoY)
                        baseForecastValue = baseValueYoY * avgYoYMultiplier;
                        forecastBasis = `YoY Base (Previous Year: ${formatValue(baseValueYoY, primaryKey)} * ${avgYoYLabel})`;
                    }
                    
                    // 5. Apply Adjustment Factors
                    let externalMultiplier = 1;
                    let internalAdditive = 0;
                    
                    // --- A. External Factors (Multiplier) ---
                    selectedExternalFactors.forEach(factor => {
                        // Apply as a multiplicative factor: (1 + factor)
                        externalMultiplier *= (1 + factor.factor);
                    });
                    
                    // --- B. Internal Factors (Additive/Duration-based) ---
                    let internalMonthlyAdjustment = 0;
                    
                    selectedInternalFactors.forEach(factor => {
                        if (factor.id === 'new_product_launch') {
                            // New Product Launch (6 Months) 
                            const forecastMonthIndex = currentPeriod - startForecastPeriodIndex + 1;
                            if (forecastMonthIndex > 0 && forecastMonthIndex <= factor.duration) {
                                // Apply the factor for the first N months (6 in this case)
                                internalMonthlyAdjustment += factor.factor;
                            }
                        } else {
                            // Other internal factors (Aggressive Marketing, Restructure) are additive to the overall factor
                            internalMonthlyAdjustment += factor.factor;
                        }
                    });
                    
                    // Final Multiplier
                    // Note: External and Internal factors are applied multiplicatively to the base forecast.
                    const totalAdjustmentMultiplier = externalMultiplier * (1 + internalMonthlyAdjustment);

                    const adjustedForecastValue = baseForecastValue * totalAdjustmentMultiplier;
                    
                    // Store totals
                    totalBaseForecast += baseForecastValue;
                    totalAdjustedForecast += adjustedForecastValue;

                    primaryValue = Math.max(0, Math.round(adjustedForecastValue));
                    
                    // Budget value for forecast months (read from CSV)
                    budgetValue = historicalPoint ? historicalPoint[budgetKey] : NaN; 
                    
                } else {
                    // Skip months that are not known historical data or part of the forecast period
                    continue;
                }

                const finalBudgetValue = isNaN(budgetValue) || budgetValue === null ? null : budgetValue;

                annualForecastData.push({
                    Month: month,
                    Year: year,
                    label: `${month.substring(0, 3)} ${year}`,
                    primaryValue: primaryValue,
                    budgetValue: finalBudgetValue,
                    isHistorical: !isForecast,
                    isForecast: isForecast,
                    forecastBasis: forecastBasis
                });
            }
            
            return {
                annualForecastData,
                totalBase: totalBaseForecast,
                totalAdjusted: totalAdjustedForecast
            };
        }

        
        // --- Chart Rendering ---

        function renderChart() {
            if (rawData.length === 0 || !DATA_KEY) return;
            
            // Check for valid historical base range
            if (BASIS_END_YEAR < FORECAST_START_YEAR || (BASIS_END_YEAR === FORECAST_START_YEAR && monthOrder.indexOf(BASIS_END_MONTH) < monthOrder.indexOf(FORECAST_START_MONTH))) {
                showMessage("Error: The Historical Base 'End Date' cannot be before the 'Start Date'. Please adjust the range.", 'error');
                return;
            }

            const { annualForecastData, totalBase, totalAdjusted } = generateForecastData(
                rawData, 
                DATA_KEY, 
                BUDGET_KEY, 
                FORECAST_START_MONTH, FORECAST_START_YEAR, BASIS_END_MONTH, BASIS_END_YEAR,
                FORECAST_DURATION_MONTHS
            );
            
            if (annualForecastData.length === 0) {
                 showMessage("Error generating forecast data. Verify your metric selection and data range.", 'error');
                 return;
            }
            
            // Calculate Avg YoY for summary display
            const { avgYoY } = calculateAverageYoY(rawData, DATA_KEY, FORECAST_START_MONTH, FORECAST_START_YEAR, BASIS_END_MONTH, BASIS_END_YEAR);
            const avgYoYLabel = (avgYoY * 100).toFixed(1) + '%';


            // --- Update Summary ---
            document.getElementById('summary-base-total').innerText = formatValue(totalBase, DATA_KEY);
            document.getElementById('summary-adjusted-total').innerText = formatValue(totalAdjusted, DATA_KEY);
            document.getElementById('calculated-yoy').innerText = avgYoYLabel + ' Average YoY';
            
            // --- Update Chart Data and UI ---
            const labels = annualForecastData.map(d => d.label);
            const primaryValues = annualForecastData.map(d => d.primaryValue);
            const budgetValues = annualForecastData.map(d => d.budgetValue);
            
            const forecastStartIndex = annualForecastData.findIndex(d => d.isForecast);
            // Color historical data green, and forecast data lighter green
            const barColors = primaryValues.map((qty, i) => i < forecastStartIndex ? '#10B981' : '#A7F3D0'); 

            const noteText = `Primary Metric: ${DATA_KEY}. Historical YoY Base: ${FORECAST_START_MONTH.substring(0, 3)} ${FORECAST_START_YEAR} to ${BASIS_END_MONTH.substring(0, 3)} ${BASIS_END_YEAR} (Average Growth: ${avgYoYLabel}). Forecast Duration: ${FORECAST_DURATION_MONTHS} Months.`;
            document.getElementById('forecast-note').innerText = noteText;
            
            const isCurrency = DATA_KEY.toLowerCase().includes('revenue') || DATA_KEY.toLowerCase().includes('sale') || DATA_KEY.toLowerCase().includes('budget');
            const currencySymbol = isCurrency ? '£' : '';
            const valueUnit = isCurrency ? '' : DATA_KEY;
            const factorCount = Array.from(document.querySelectorAll('#external-factors-list input:checked, #internal-factors-list input:checked')).length;
            document.getElementById('chart-title').innerText = `Monthly Trend: ${DATA_KEY} (Base YoY Model - ${factorCount} Factor(s) Applied)`;


            const ctx = document.getElementById('mainChart').getContext('2d');
            if (mainChart) {
                mainChart.destroy();
            }

            mainChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [
                         {
                            label: BUDGET_KEY || 'Budget/Target (N/A)',
                            data: budgetValues,
                            type: 'line',
                            borderColor: '#3B82F6', 
                            borderWidth: 3,
                            pointRadius: 3,
                            pointBackgroundColor: '#3B82F6',
                            tension: 0.1,
                            order: 1, 
                        },
                        {
                            label: `Actual / Adjusted Forecast ${DATA_KEY}`,
                            data: primaryValues,
                            backgroundColor: barColors,
                            borderColor: '#059669',
                            borderWidth: 1,
                            order: 2, 
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: true,
                    interaction: {
                        mode: 'index',
                        intersect: false,
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            title: {
                                display: true,
                                text: DATA_KEY + (valueUnit ? ` (${valueUnit})` : '')
                            },
                            ticks: {
                                callback: function(value) {
                                    return currencySymbol + value.toLocaleString('en-US', { minimumFractionDigits: 0, maximumFractionDigits: 0 });
                                }
                            }
                        },
                        x: {
                            title: {
                                display: true,
                                text: 'Month'
                            }
                        }
                    },
                    plugins: {
                        legend: {
                            display: true,
                            labels: {
                                usePointStyle: true
                            }
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    let label = context.dataset.label || '';
                                    if (label) {
                                        label += ': ';
                                    }
                                    if (context.parsed.y !== null) {
                                        label += formatValue(context.parsed.y, context.dataset.label);
                                    }
                                    return label;
                                },
                                afterLabel: function(context) {
                                    const dataIndex = context.dataIndex;
                                    const pointData = annualForecastData[dataIndex];
                                    
                                    if (context.dataset.label.includes(DATA_KEY) && pointData) {
                                        if (pointData.isForecast) {
                                            return `Model Basis: ${pointData.forecastBasis}`;
                                        }
                                    }
                                    return null;
                                }
                            }
                        }
                    }
                }
            });
        }
        
        // --- UI Population and Event Handling ---
        
        /** Populates metric selectors dynamically. */
        function populateMetricSelectors() {
            const dataSelect = document.getElementById('data-metric-selector');
            const budgetSelect = document.getElementById('budget-metric-selector');
            
            dataSelect.innerHTML = '';
            budgetSelect.innerHTML = '';
            
            const noneOption = `<option value="">-- None --</option>`;
            budgetSelect.insertAdjacentHTML('beforeend', noneOption);

            let firstMetric = null;

            numericHeaders.forEach(header => {
                const option = `<option value="${header}">${header}</option>`;
                
                dataSelect.insertAdjacentHTML('beforeend', option);
                budgetSelect.insertAdjacentHTML('beforeend', option);
                
                if (!firstMetric) {
                    firstMetric = header;
                }
            });
            
            if (firstMetric) {
                dataSelect.value = firstMetric;
                DATA_KEY = firstMetric;
                
                const budgetGuess = numericHeaders.find(h => h.toLowerCase().includes('budget'));
                if (budgetGuess) {
                    budgetSelect.value = budgetGuess;
                    BUDGET_KEY = budgetGuess;
                } else {
                    BUDGET_KEY = '';
                }
            }
        }
        
        /** Populates historical base date selectors, defaulting to the last 6 months. */
        function populateDateSelectors() {
            const startBasisSelect = document.getElementById('forecast-start-selector');
            const endBasisSelect = document.getElementById('forecast-basis-end-selector');
            
            startBasisSelect.innerHTML = '';
            endBasisSelect.innerHTML = '';
            
            // Get the last historical point with data for the selected metric
            let lastHistoricalPoint = getLastHistoricalPoint(rawData, DATA_KEY || numericHeaders[0]);
            
            if (!lastHistoricalPoint) return;
            
            // 1. Collect all valid historical dates
            const historicalDates = [];
            rawData.forEach(d => {
                 const monthIndex = monthOrder.indexOf(d.Month);
                 const currentPeriod = d.Year * 12 + monthIndex;
                 const historicalPeriod = lastHistoricalPoint.Year * 12 + monthOrder.indexOf(lastHistoricalPoint.Month);
                 
                 // Only include months up to the last historical point
                 if (currentPeriod <= historicalPeriod) { 
                    historicalDates.push({ month: d.Month, year: d.Year, period: currentPeriod });
                 }
            });
            
            historicalDates.sort((a, b) => a.period - b.period);

            historicalDates.forEach((d, index) => {
                const value = `${d.month}|${d.year}`;
                const label = `${d.month} ${d.year}`;
                const option = `<option value="${value}">${label}</option>`;
                
                startBasisSelect.insertAdjacentHTML('beforeend', option);
                endBasisSelect.insertAdjacentHTML('beforeend', option);
            });
            
            // 2. --- Set Default Base Period (LAST 6 MONTHS) ---

            // Set default for End Basis (Latest historical date available)
            const lastIndex = historicalDates.length - 1;
            if (lastIndex >= 0) {
                const lastDate = historicalDates[lastIndex];
                BASIS_END_MONTH = lastDate.month;
                BASIS_END_YEAR = lastDate.year;
                endBasisSelect.value = `${lastDate.month}|${lastDate.year}`;
                
                // Set default for Start Basis (6 months prior to the end date, or the first available date)
                // Index 5 before the last one = 6 months total in range (index [0] to index [5])
                const defaultStartIndex = Math.max(0, lastIndex - 5); 
                const defaultStart = historicalDates[defaultStartIndex];
                
                FORECAST_START_MONTH = defaultStart.month;
                FORECAST_START_YEAR = defaultStart.year;
                startBasisSelect.value = `${defaultStart.month}|${defaultStart.year}`;
            }
        }
        
        /** Populates the factor checkbox lists. */
        function populateFactorSelectors() {
            const externalList = document.getElementById('external-factors-list');
            const internalList = document.getElementById('internal-factors-list');

            externalList.innerHTML = '';
            internalList.innerHTML = '';
            
            const createFactorHtml = (factor) => {
                const isNegative = factor.factor < 0;
                const sign = isNegative ? '' : '+';
                const color = isNegative ? 'text-red-600' : 'text-green-600';
                
                let durationNote = '';
                if (factor.duration) {
                    durationNote = ` (Only the first ${factor.duration} forecast months)`;
                }

                return `
                    <label for="factor_${factor.id}" class="flex items-center space-x-3 factor-label cursor-pointer p-2 rounded-lg transition duration-150 ${isNegative ? 'bg-red-100 hover:bg-red-200' : 'bg-green-100 hover:bg-green-200'}">
                        <input type="checkbox" id="factor_${factor.id}" value="${factor.id}" class="form-checkbox h-5 w-5 ${isNegative ? 'text-red-600' : 'text-green-600'} rounded border-gray-300 focus:ring-0">
                        <span class="text-sm text-gray-800">${factor.label}${durationNote}</span>
                        <span class="text-sm font-semibold ${color}">(${sign}${(factor.factor * 100).toFixed(0)}%)</span>
                    </label>
                `;
            };

            FACTORS_CONFIG.external.forEach(factor => {
                externalList.insertAdjacentHTML('beforeend', createFactorHtml(factor));
            });

            FACTORS_CONFIG.internal.forEach(factor => {
                internalList.insertAdjacentHTML('beforeend', createFactorHtml(factor));
            });
            
            // Add listeners for factor changes
            document.querySelectorAll('#external-factors-list input, #internal-factors-list input').forEach(checkbox => {
                checkbox.addEventListener('change', handleSelectorChange);
            });
        }

        // --- Main Listener Function ---
        
        function handleSelectorChange() {
            // Update selected values from UI
            DATA_KEY = document.getElementById('data-metric-selector').value;
            BUDGET_KEY = document.getElementById('budget-metric-selector').value;
            FORECAST_DURATION_MONTHS = parseInt(document.getElementById('forecast-duration-selector').value);
            
            if (document.getElementById('forecast-start-selector').value) {
                const [month, year] = document.getElementById('forecast-start-selector').value.split('|');
                FORECAST_START_MONTH = month;
                FORECAST_START_YEAR = parseInt(year);
            }

            if (document.getElementById('forecast-basis-end-selector').value) {
                const [month, year] = document.getElementById('forecast-basis-end-selector').value.split('|');
                BASIS_END_MONTH = month;
                BASIS_END_YEAR = parseInt(year);
            }
            
            // Re-render chart if data is loaded
            if (rawData.length > 0 && DATA_KEY) {
                renderChart();
            }
        }

        // --- DOM Ready Event Listener ---
        
        document.addEventListener('DOMContentLoaded', () => {
            const fileInput = document.getElementById('csv-file');
            const dropzone = document.getElementById('dropzone');
            const exportBtn = document.getElementById('export-chart-btn');
            
            // Selectors listeners
            document.getElementById('data-metric-selector').addEventListener('change', () => {
                populateDateSelectors(); // Re-populate dates when metric changes
                handleSelectorChange();
            });
            document.getElementById('budget-metric-selector').addEventListener('change', handleSelectorChange);
            document.getElementById('forecast-duration-selector').addEventListener('change', handleSelectorChange);
            document.getElementById('forecast-start-selector').addEventListener('change', handleSelectorChange);
            document.getElementById('forecast-basis-end-selector').addEventListener('change', handleSelectorChange);

            populateFactorSelectors(); // Initial factor population

            // --- Drag & Drop and Click Events ---
            dropzone.addEventListener('click', () => fileInput.click());
            dropzone.addEventListener('dragover', (e) => {
                e.preventDefault();
                dropzone.classList.add('hover');
            });
            dropzone.addEventListener('dragleave', (e) => {
                e.preventDefault();
                dropzone.classList.remove('hover');
            });
            dropzone.addEventListener('drop', (e) => {
                e.preventDefault();
                dropzone.classList.remove('hover');
                if (e.dataTransfer.files.length) {
                    handleFileUpload(e.dataTransfer.files[0]);
                }
            });
            fileInput.addEventListener('change', (e) => {
                if (e.target.files.length) { 
                    handleFileUpload(e.target.files[0]);
                }
            });

            // File Upload Handler
            function handleFileUpload(file) {
                if (file.type !== 'text/csv' && !file.name.endsWith('.csv')) {
                    showMessage('Error: Please upload a file with the .csv extension.', 'error');
                    return;
                }

                const reader = new FileReader();
                reader.onload = (e) => {
                    const csvText = e.target.result;
                    const parsedData = parseCSV(csvText);
                    
                    if (parsedData) {
                        rawData = parsedData;
                        
                        // 1. Populate dynamic selectors
                        populateMetricSelectors();
                        populateDateSelectors();
                        
                        // 2. Set initial state and render
                        handleSelectorChange(); 
                        
                        // 3. Update UI
                        document.getElementById('file-name').innerText = file.name;
                        document.getElementById('file-status').classList.remove('hidden');
                        document.getElementById('chart-container').classList.remove('hidden');
                        document.getElementById('controls-container').classList.remove('hidden');
                        showMessage(`File '${file.name}' processed successfully. Define your historical base and select factors.`, 'success');
                    } else {
                        // Reset UI if parsing failed
                        rawData = [];
                        document.getElementById('chart-container').classList.add('hidden');
                        document.getElementById('controls-container').classList.add('hidden');
                    }
                };
                reader.onerror = () => {
                    showMessage('Error reading file. Please try again.', 'error');
                };
                reader.readAsText(file);
            }

            // Chart Export Event
            exportBtn.addEventListener('click', () => {
                const canvas = document.getElementById('mainChart');
                
                if (canvas && mainChart) {
                    const a = document.createElement('a');
                    a.href = mainChart.toBase64Image();
                    const duration = FORECAST_DURATION_MONTHS;
                    const factorCount = Array.from(document.querySelectorAll('#external-factors-list input:checked, #internal-factors-list input:checked')).length;
                    a.download = `${DATA_KEY.toLowerCase()}_forecast_${duration}mo_${factorCount}factors.png`;
                    document.body.appendChild(a);
                    a.click();
                    document.body.removeChild(a);
                    showMessage('Chart exported as PNG.', 'success');
                } else {
                    showMessage('Error: No chart to export. Load a CSV file first.', 'error');
                }
            });
        });
        
    </script>
</body>
</html>
