 <!DOCTYPE html>
<html>
<head>
    <title>Number Pattern Analyzer</title>
    <style>
        body { font-family: Arial, sans-serif; max-width: 900px; margin: 0 auto; padding: 20px; }
        .input-area { background: #f5f5f5; padding: 15px; margin: 15px 0; border-radius: 5px; }
        textarea { 
            width: 100%; padding: 10px; margin: 5px 0; 
            font-family: monospace; height: 100px;
        }
        button { 
            background: #3498db; color: white; border: none; padding: 10px 15px; 
            border-radius: 4px; cursor: pointer; margin: 5px; font-size: 16px;
        }
        button:hover { background: #2980b9; }
        .digit { 
            display: inline-block; width: 35px; height: 35px; background: #4CAF50; 
            color: white; text-align: center; line-height: 35px; border-radius: 50%; 
            margin: 2px; font-weight: bold; font-size: 16px;
        }
        .number { 
            display: inline-block; padding: 5px 10px; background: #2196F3; 
            color: white; text-align: center; border-radius: 4px; 
            margin: 2px; font-weight: bold;
        }
        .results { margin-top: 20px; padding: 20px; background: #f9f9f9; border-radius: 5px; }
        .tab { overflow: hidden; border: 1px solid #ccc; background-color: #f1f1f1; }
        .tab button { 
            background-color: inherit; float: left; border: none; outline: none; 
            cursor: pointer; padding: 12px 16px; transition: 0.3s; color: black;
            font-size: 15px;
        }
        .tab button.active { background-color: #3498db; color: white; }
        .tabcontent { display: none; padding: 20px; border: 1px solid #ccc; border-top: none; }
        .prediction { margin: 20px 0; padding: 15px; background: white; border-radius: 5px; }
        .instructions { background: #fffde7; padding: 15px; border-radius: 5px; margin-bottom: 20px; }
        .frequency-table { width: 100%; border-collapse: collapse; margin: 15px 0; }
        .frequency-table th, .frequency-table td { 
            border: 1px solid #ddd; padding: 8px; text-align: center; 
        }
        .frequency-table th { background-color: #f2f2f2; }
        .option-group { margin: 10px 0; }
        .number-list { margin: 15px 0; }
        .ad-space { 
            background: #f0f0f0; border: 1px dashed #ccc; padding: 15px; 
            text-align: center; margin: 20px 0; min-height: 90px;
        }
    </style>
</head>
<body>
    <div class="ad-space">
        <p>Advertisement Space</p>
        <div style="color:#888">Your ad could appear here</div>
    </div>
    
    <h1>Number Pattern Analyzer</h1>
    
    <div class="instructions">
        <h3>How to Use:</h3>
        <p><strong>With spaces:</strong> Numbers are read as separate values (e.g., "12 34 56" becomes [12, 34, 56])</p>
        <p><strong>Without spaces:</strong> Numbers are split into individual digits (e.g., "123456" becomes [1,2,3,4,5,6])</p>
        <p>At least 10 numbers required for meaningful analysis</p>
    </div>
    
    <div class="input-area">
        <textarea id="numberInput" placeholder="Enter numbers here...
With spaces (for multi-digit numbers):
12 34 56 78 90

Without spaces (for single-digit analysis):
1234567890"></textarea>
        
        <div class="option-group">
            <label>
                <input type="checkbox" id="removeSpaces"> 
                Remove spaces between numbers (analyze as individual digits)
            </label>
        </div>
        
        <button id="analyze">Analyze Numbers</button>
        <button id="clear">Clear All</button>
    </div>
    
    <div class="ad-space">
        <p>Advertisement Space</p>
        <div style="color:#888">Your ad could appear here</div>
    </div>
    
    <div id="results" class="results" style="display: none;">
        <h2>Analysis Results</h2>
        
        <div class="number-list">
            <h3>Numbers Being Analyzed:</h3>
            <div id="numbersDisplay"></div>
        </div>
        
        <h3>Digit Frequency (0-9)</h3>
        <table class="frequency-table" id="frequencyTable">
            <thead>
                <tr>
                    <th>Digit</th><th>Count</th><th>Percentage</th>
                </tr>
            </thead>
            <tbody id="frequencyBody">
            </tbody>
        </table>
        
        <div class="tab">
            <button class="tablinks active" onclick="openTab(event, 'monteCarloTab')">Monte Carlo</button>
            <button class="tablinks" onclick="openTab(event, 'markovTab')">Markov Chain</button>
        </div>
        
        <div id="monteCarloTab" class="tabcontent" style="display: block;">
            <div id="monteCarloResults"></div>
        </div>
        
        <div id="markovTab" class="tabcontent">
            <div id="markovResults"></div>
        </div>
    </div>

    <div class="ad-space">
        <p>Advertisement Space</p>
        <div style="color:#888">Your ad could appear here</div>
    </div>

    <script>
        // Global variables
        let numbers = [];
        let digitCounts = Array(10).fill(0);
        
        // DOM elements
        const numberInput = document.getElementById('numberInput');
        const removeSpaces = document.getElementById('removeSpaces');
        const analyzeBtn = document.getElementById('analyze');
        const clearBtn = document.getElementById('clear');
        const resultsDiv = document.getElementById('results');
        const frequencyBody = document.getElementById('frequencyBody');
        const numbersDisplay = document.getElementById('numbersDisplay');
        
        // Initialize
        document.addEventListener('DOMContentLoaded', function() {
            analyzeBtn.addEventListener('click', analyzeData);
            clearBtn.addEventListener('click', clearAll);
        });
        
        // Tab functionality
        function openTab(evt, tabName) {
            const tabcontent = document.getElementsByClassName("tabcontent");
            for (let i = 0; i < tabcontent.length; i++) {
                tabcontent[i].style.display = "none";
            }
            
            const tablinks = document.getElementsByClassName("tablinks");
            for (let i = 0; i < tablinks.length; i++) {
                tablinks[i].className = tablinks[i].className.replace(" active", "");
            }
            
            document.getElementById(tabName).style.display = "block";
            evt.currentTarget.className += " active";
        }
        
        // Clear all inputs
        function clearAll() {
            numberInput.value = '';
            numbers = [];
            digitCounts = Array(10).fill(0);
            resultsDiv.style.display = 'none';
        }
        
        // Parse number input
        function parseInput() {
            let input = numberInput.value.trim();
            if (!input) return [];
            
            if (removeSpaces.checked) {
                // Remove all spaces and treat as individual digits
                input = input.replace(/\s+/g, '');
                return input.split('').filter(c => /\d/.test(c)).map(Number);
            } else {
                // Treat space-separated values as complete numbers
                return input.split(/\s+/).map(Number).filter(n => !isNaN(n));
            }
        }
        
        // Calculate digit frequencies
        function calculateFrequencies(nums) {
            const counts = Array(10).fill(0);
            if (removeSpaces.checked) {
                // Count individual digits
                nums.forEach(d => {
                    if (d >= 0 && d <= 9) counts[d]++;
                });
            } else {
                // Count digits within multi-digit numbers
                nums.forEach(num => {
                    num.toString().split('').forEach(d => {
                        const digit = parseInt(d);
                        if (digit >= 0 && digit <= 9) counts[digit]++;
                    });
                });
            }
            return counts;
        }
        
        // Display the numbers being analyzed
        function displayNumbers(nums) {
            numbersDisplay.innerHTML = '';
            if (removeSpaces.checked) {
                nums.forEach(num => {
                    numbersDisplay.innerHTML += `<span class="digit">${num}</span>`;
                });
            } else {
                nums.forEach(num => {
                    numbersDisplay.innerHTML += `<span class="number">${num}</span>`;
                });
            }
        }
        
        // Display frequency table
        function displayFrequencies(counts) {
            const total = counts.reduce((a, b) => a + b, 0);
            frequencyBody.innerHTML = '';
            
            for (let i = 0; i < 10; i++) {
                const percentage = total > 0 ? (counts[i] / total * 100).toFixed(2) + '%' : '0%';
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td><span class="digit">${i}</span></td>
                    <td>${counts[i]}</td>
                    <td>${percentage}</td>
                `;
                frequencyBody.appendChild(row);
            }
        }
        
        // Main analysis function
        function analyzeData() {
            numbers = parseInput();
            
            if (numbers.length < 10) {
                alert('Please enter at least 10 numbers for meaningful analysis');
                return;
            }
            
            // Display the numbers being analyzed
            displayNumbers(numbers);
            
            // Calculate digit frequencies
            digitCounts = calculateFrequencies(numbers);
            displayFrequencies(digitCounts);
            
            // Run Monte Carlo simulation
            const monteCarloResults = runMonteCarlo(numbers, 1000);
            displayMonteCarloResults(monteCarloResults);
            
            // Run Markov Chain prediction
            const markovResults = runMarkovChain(numbers);
            displayMarkovResults(markovResults);
            
            // Show results
            resultsDiv.style.display = 'block';
        }
        
        // Monte Carlo simulation
        function runMonteCarlo(nums, iterations) {
            // Create weighted array based on analysis mode
            let weightedItems = [];
            
            if (removeSpaces.checked) {
                // Weight by individual digit frequency
                digitCounts.forEach((count, digit) => {
                    for (let i = 0; i < count; i++) {
                        weightedItems.push(digit);
                    }
                });
            } else {
                // Weight by complete number frequency
                const numberCounts = {};
                nums.forEach(num => {
                    numberCounts[num] = (numberCounts[num] || 0) + 1;
                });
                
                Object.entries(numberCounts).forEach(([num, count]) => {
                    for (let i = 0; i < count; i++) {
                        weightedItems.push(parseInt(num));
                    }
                });
            }
            
            // If no weights (all counts zero), use equal probability
            if (weightedItems.length === 0) {
                if (removeSpaces.checked) {
                    for (let i = 0; i < 10; i++) weightedItems.push(i);
                } else {
                    // For multi-digit, we can't reasonably create equal probability
                    return [];
                }
            }
            
            // Run simulations
            const results = {};
            const predictionLength = removeSpaces.checked ? 5 : Math.max(1, Math.floor(Math.log10(nums[0])) + 1);
            
            for (let i = 0; i < iterations; i++) {
                // Create a prediction
                let prediction = [];
                for (let j = 0; j < predictionLength; j++) {
                    const item = weightedItems[Math.floor(Math.random() * weightedItems.length)];
                    prediction.push(item);
                }
                
                // Store prediction
                const key = prediction.join('');
                results[key] = (results[key] || 0) + 1;
            }
            
            // Sort by frequency
            const sorted = Object.entries(results)
                .sort((a, b) => b[1] - a[1])
                .slice(0, 5)
                .map(([pred, count]) => ({
                    prediction: pred.split('').map(Number),
                    count
                }));
            
            return sorted;
        }
        
        // Display Monte Carlo results
        function displayMonteCarloResults(results) {
            let html = '<div class="prediction">';
            html += `<h3>Monte Carlo Predictions (${removeSpaces.checked ? 'single digits' : 'multi-digit numbers'})</h3>`;
            html += '<p>Based on 1000 simulations weighted by frequency</p>';
            
            if (results.length === 0) {
                html += '<p>No valid predictions generated</p>';
            } else {
                html += '<ol>';
                results.forEach((result, index) => {
                    html += '<li>';
                    if (removeSpaces.checked) {
                        html += result.prediction.map(d => `<span class="digit">${d}</span>`).join(' ');
                    } else {
                        html += `<span class="number">${result.prediction.join('')}</span>`;
                    }
                    html += ` (appeared ${result.count} times in simulation)</li>`;
                });
                html += '</ol>';
            }
            
            html += '</div>';
            document.getElementById('monteCarloResults').innerHTML = html;
        }
        
        // Markov Chain prediction
        function runMarkovChain(nums) {
            if (nums.length < 2) return [];
            
            if (removeSpaces.checked) {
                // Single-digit Markov analysis
                const matrix = Array(10).fill().map(() => Array(10).fill(0));
                
                // Count transitions between digits
                for (let i = 1; i < nums.length; i++) {
                    const prev = nums[i-1];
                    const current = nums[i];
                    matrix[prev][current]++;
                }
                
                // Get probabilities for next digits
                const lastDigit = nums[nums.length-1];
                const probs = matrix[lastDigit].map((count, digit) => ({
                    value: digit,
                    count
                }));
                
                return probs.sort((a, b) => b.count - a.count)
                           .slice(0, 5)
                           .filter(item => item.count > 0);
            } else {
                // Multi-digit Markov analysis
                const numberSet = [...new Set(nums)];
                const matrix = Array(numberSet.length).fill().map(() => Array(numberSet.length).fill(0));
                
                // Create mapping between numbers and indices
                const numToIndex = {};
                numberSet.forEach((num, index) => {
                    numToIndex[num] = index;
                });
                
                // Count transitions between numbers
                for (let i = 1; i < nums.length; i++) {
                    const prevIndex = numToIndex[nums[i-1]];
                    const currentIndex = numToIndex[nums[i]];
                    matrix[prevIndex][currentIndex]++;
                }
                
                // Get probabilities for next numbers
                const lastNum = nums[nums.length-1];
                const lastIndex = numToIndex[lastNum];
                const probs = matrix[lastIndex].map((count, index) => ({
                    value: numberSet[index],
                    count
                }));
                
                return probs.sort((a, b) => b.count - a.count)
                           .slice(0, 5)
                           .filter(item => item.count > 0);
            }
        }
        
        // Display Markov results
        function displayMarkovResults(results) {
            let html = '<div class="prediction">';
            html += `<h3>Markov Chain Predictions (${removeSpaces.checked ? 'next digit' : 'next number'})</h3>`;
            html += '<p>Most likely values based on transition probabilities</p>';
            
            if (results.length === 0) {
                html += '<p>No significant patterns detected</p>';
            } else {
                html += '<ol>';
                results.forEach((result, index) => {
                    html += '<li>';
                    if (removeSpaces.checked) {
                        html += `<span class="digit">${result.value}</span> `;
                    } else {
                        html += `<span class="number">${result.value}</span> `;
                    }
                    html += `(transition count: ${result.count})</li>`;
                });
                html += '</ol>';
            }
            
            html += '</div>';
            document.getElementById('markovResults').innerHTML = html;
        }
    </script>
</body>
</html>
           cursor: pointer; padding: 12px 16px; transition: 0.3s; color: black;
            font-size: 15px;
        }
        .tab button.active { background-color: #3498db; color: white; }
        .tabcontent { display: none; padding: 20px; border: 1px solid #ccc; border-top: none; }
        .prediction { margin: 20px 0; padding: 15px; background: white; border-radius: 5px; }
        .instructions { background: #fffde7; padding: 15px; border-radius: 5px; margin-bottom: 20px; }
        .frequency-table { width: 100%; border-collapse: collapse; margin: 15px 0; }
        .frequency-table th, .frequency-table td { 
            border: 1px solid #ddd; padding: 8px; text-align: center; 
        }
        .frequency-table th { background-color: #f2f2f2; }
        .option-group { margin: 10px 0; }
        .number-list { margin: 15px 0; }
        .ad-space { 
            background: #f0f0f0; border: 1px dashed #ccc; padding: 15px; 
            text-align: center; margin: 20px 0; min-height: 90px;
        }
    </style>
</head>
<body>
    <div class="ad-space">
        <p>Advertisement Space</p>
        <div style="color:#888">Your ad could appear here</div>
    </div>
    
    <h1>Number Pattern Analyzer</h1>
    
    <div class="instructions">
        <h3>How to Use:</h3>
        <p><strong>With spaces:</strong> Numbers are read as separate values (e.g., "12 34 56" becomes [12, 34, 56])</p>
        <p><strong>Without spaces:</strong> Numbers are split into individual digits (e.g., "123456" becomes [1,2,3,4,5,6])</p>
        <p>At least 10 numbers required for meaningful analysis</p>
    </div>
    
    <div class="input-area">
        <textarea id="numberInput" placeholder="Enter numbers here...
With spaces (for multi-digit numbers):
12 34 56 78 90

Without spaces (for single-digit analysis):
1234567890"></textarea>
        
        <div class="option-group">
            <label>
                <input type="checkbox" id="removeSpaces"> 
                Remove spaces between numbers (analyze as individual digits)
            </label>
        </div>
        
        <button id="analyze">Analyze Numbers</button>
        <button id="clear">Clear All</button>
    </div>
    
    <div class="ad-space">
        <p>Advertisement Space</p>
        <div style="color:#888">Your ad could appear here</div>
    </div>
    
    <div id="results" class="results" style="display: none;">
        <h2>Analysis Results</h2>
        
        <div class="number-list">
            <h3>Numbers Being Analyzed:</h3>
            <div id="numbersDisplay"></div>
        </div>
        
        <h3>Digit Frequency (0-9)</h3>
        <table class="frequency-table" id="frequencyTable">
            <thead>
                <tr>
                    <th>Digit</th><th>Count</th><th>Percentage</th>
                </tr>
            </thead>
            <tbody id="frequencyBody">
            </tbody>
        </table>
        
        <div class="tab">
            <button class="tablinks active" onclick="openTab(event, 'monteCarloTab')">Monte Carlo</button>
            <button class="tablinks" onclick="openTab(event, 'markovTab')">Markov Chain</button>
        </div>
        
        <div id="monteCarloTab" class="tabcontent" style="display: block;">
            <div id="monteCarloResults"></div>
        </div>
        
        <div id="markovTab" class="tabcontent">
            <div id="markovResults"></div>
        </div>
    </div>

    <div class="ad-space">
        <p>Advertisement Space</p>
        <div style="color:#888">Your ad could appear here</div>
    </div>

    <script>
        // Global variables
        let numbers = [];
        let digitCounts = Array(10).fill(0);
        
        // DOM elements
        const numberInput = document.getElementById('numberInput');
        const removeSpaces = document.getElementById('removeSpaces');
        const analyzeBtn = document.getElementById('analyze');
        const clearBtn = document.getElementById('clear');
        const resultsDiv = document.getElementById('results');
        const frequencyBody = document.getElementById('frequencyBody');
        const numbersDisplay = document.getElementById('numbersDisplay');
        
        // Initialize
        document.addEventListener('DOMContentLoaded', function() {
            analyzeBtn.addEventListener('click', analyzeData);
            clearBtn.addEventListener('click', clearAll);
        });
        
        // Tab functionality
        function openTab(evt, tabName) {
            const tabcontent = document.getElementsByClassName("tabcontent");
            for (let i = 0; i < tabcontent.length; i++) {
                tabcontent[i].style.display = "none";
            }
            
            const tablinks = document.getElementsByClassName("tablinks");
            for (let i = 0; i < tablinks.length; i++) {
                tablinks[i].className = tablinks[i].className.replace(" active", "");
            }
            
            document.getElementById(tabName).style.display = "block";
            evt.currentTarget.className += " active";
        }
        
        // Clear all inputs
        function clearAll() {
            numberInput.value = '';
            numbers = [];
            digitCounts = Array(10).fill(0);
            resultsDiv.style.display = 'none';
        }
        
        // Parse number input
        function parseInput() {
            let input = numberInput.value.trim();
            if (!input) return [];
            
            if (removeSpaces.checked) {
                // Remove all spaces and treat as individual digits
                input = input.replace(/\s+/g, '');
                return input.split('').filter(c => /\d/.test(c)).map(Number);
            } else {
                // Treat space-separated values as complete numbers
                return input.split(/\s+/).map(Number).filter(n => !isNaN(n));
            }
        }
        
        // Calculate digit frequencies
        function calculateFrequencies(nums) {
            const counts = Array(10).fill(0);
            if (removeSpaces.checked) {
                // Count individual digits
                nums.forEach(d => {
                    if (d >= 0 && d <= 9) counts[d]++;
                });
            } else {
                // Count digits within multi-digit numbers
                nums.forEach(num => {
                    num.toString().split('').forEach(d => {
                        const digit = parseInt(d);
                        if (digit >= 0 && digit <= 9) counts[digit]++;
                    });
                });
            }
            return counts;
        }
        
        // Display the numbers being analyzed
        function displayNumbers(nums) {
            numbersDisplay.innerHTML = '';
            if (removeSpaces.checked) {
                nums.forEach(num => {
                    numbersDisplay.innerHTML += `<span class="digit">${num}</span>`;
                });
            } else {
                nums.forEach(num => {
                    numbersDisplay.innerHTML += `<span class="number">${num}</span>`;
                });
            }
        }
        
        // Display frequency table
        function displayFrequencies(counts) {
            const total = counts.reduce((a, b) => a + b, 0);
            frequencyBody.innerHTML = '';
            
            for (let i = 0; i < 10; i++) {
                const percentage = total > 0 ? (counts[i] / total * 100).toFixed(2) + '%' : '0%';
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td><span class="digit">${i}</span></td>
                    <td>${counts[i]}</td>
                    <td>${percentage}</td>
                `;
                frequencyBody.appendChild(row);
            }
        }
        
        // Main analysis function
        function analyzeData() {
            numbers = parseInput();
            
            if (numbers.length < 10) {
                alert('Please enter at least 10 numbers for meaningful analysis');
                return;
            }
            
            // Display the numbers being analyzed
            displayNumbers(numbers);
            
            // Calculate digit frequencies
            digitCounts = calculateFrequencies(numbers);
            displayFrequencies(digitCounts);
            
            // Run Monte Carlo simulation
            const monteCarloResults = runMonteCarlo(numbers, 1000);
            displayMonteCarloResults(monteCarloResults);
            
            // Run Markov Chain prediction
            const markovResults = runMarkovChain(numbers);
            displayMarkovResults(markovResults);
            
            // Show results
            resultsDiv.style.display = 'block';
        }
        
        // Monte Carlo simulation
        function runMonteCarlo(nums, iterations) {
            // Create weighted array based on analysis mode
            let weightedItems = [];
            
            if (removeSpaces.checked) {
                // Weight by individual digit frequency
                digitCounts.forEach((count, digit) => {
                    for (let i = 0; i < count; i++) {
                        weightedItems.push(digit);
                    }
                });
            } else {
                // Weight by complete number frequency
                const numberCounts = {};
                nums.forEach(num => {
                    numberCounts[num] = (numberCounts[num] || 0) + 1;
                });
                
                Object.entries(numberCounts).forEach(([num, count]) => {
                    for (let i = 0; i < count; i++) {
                        weightedItems.push(parseInt(num));
                    }
                });
            }
            
            // If no weights (all counts zero), use equal probability
            if (weightedItems.length === 0) {
                if (removeSpaces.checked) {
                    for (let i = 0; i < 10; i++) weightedItems.push(i);
                } else {
                    // For multi-digit, we can't reasonably create equal probability
                    return [];
                }
            }
            
            // Run simulations
            const results = {};
            const predictionLength = removeSpaces.checked ? 5 : Math.max(1, Math.floor(Math.log10(nums[0])) + 1);
            
            for (let i = 0; i < iterations; i++) {
                // Create a prediction
                let prediction = [];
                for (let j = 0; j < predictionLength; j++) {
                    const item = weightedItems[Math.floor(Math.random() * weightedItems.length)];
                    prediction.push(item);
                }
                
                // Store prediction
                const key = prediction.join('');
                results[key] = (results[key] || 0) + 1;
            }
            
            // Sort by frequency
            const sorted = Object.entries(results)
                .sort((a, b) => b[1] - a[1])
                .slice(0, 5)
                .map(([pred, count]) => ({
                    prediction: pred.split('').map(Number),
                    count
                }));
            
            return sorted;
        }
        
        // Display Monte Carlo results
        function displayMonteCarloResults(results) {
            let html = '<div class="prediction">';
            html += `<h3>Monte Carlo Predictions (${removeSpaces.checked ? 'single digits' : 'multi-digit numbers'})</h3>`;
            html += '<p>Based on 1000 simulations weighted by frequency</p>';
            
            if (results.length === 0) {
                html += '<p>No valid predictions generated</p>';
            } else {
                html += '<ol>';
                results.forEach((result, index) => {
                    html += '<li>';
                    if (removeSpaces.checked) {
                        html += result.prediction.map(d => `<span class="digit">${d}</span>`).join(' ');
                    } else {
                        html += `<span class="number">${result.prediction.join('')}</span>`;
                    }
                    html += ` (appeared ${result.count} times in simulation)</li>`;
                });
                html += '</ol>';
            }
            
            html += '</div>';
            document.getElementById('monteCarloResults').innerHTML = html;
        }
        
        // Markov Chain prediction
        function runMarkovChain(nums) {
            if (nums.length < 2) return [];
            
            if (removeSpaces.checked) {
                // Single-digit Markov analysis
                const matrix = Array(10).fill().map(() => Array(10).fill(0));
                
                // Count transitions between digits
                for (let i = 1; i < nums.length; i++) {
                    const prev = nums[i-1];
                    const current = nums[i];
                    matrix[prev][current]++;
                }
                
                // Get probabilities for next digits
                const lastDigit = nums[nums.length-1];
                const probs = matrix[lastDigit].map((count, digit) => ({
                    value: digit,
                    count
                }));
                
                return probs.sort((a, b) => b.count - a.count)
                           .slice(0, 5)
                           .filter(item => item.count > 0);
            } else {
                // Multi-digit Markov analysis
                const numberSet = [...new Set(nums)];
                const matrix = Array(numberSet.length).fill().map(() => Array(numberSet.length).fill(0));
                
                // Create mapping between numbers and indices
                const numToIndex = {};
                numberSet.forEach((num, index) => {
                    numToIndex[num] = index;
                });
                
                // Count transitions between numbers
                for (let i = 1; i < nums.length; i++) {
                    const prevIndex = numToIndex[nums[i-1]];
                    const currentIndex = numToIndex[nums[i]];
                    matrix[prevIndex][currentIndex]++;
                }
                
                // Get probabilities for next numbers
                const lastNum = nums[nums.length-1];
                const lastIndex = numToIndex[lastNum];
                const probs = matrix[lastIndex].map((count, index) => ({
                    value: numberSet[index],
                    count
                }));
                
                return probs.sort((a, b) => b.count - a.count)
                           .slice(0, 5)
                           .filter(item => item.count > 0);
            }
        }
        
        // Display Markov results
        function displayMarkovResults(results) {
            let html = '<div class="prediction">';
            html += `<h3>Markov Chain Predictions (${removeSpaces.checked ? 'next digit' : 'next number'})</h3>`;
            html += '<p>Most likely values based on transition probabilities</p>';
            
            if (results.length === 0) {
                html += '<p>No significant patterns detected</p>';
            } else {
                html += '<ol>';
                results.forEach((result, index) => {
                    html += '<li>';
                    if (removeSpaces.checked) {
                        html += `<span class="digit">${result.value}</span> `;
                    } else {
                        html += `<span class="number">${result.value}</span> `;
                    }
                    html += `(transition count: ${result.count})</li>`;
                });
                html += '</ol>';
            }
            
            html += '</div>';
            document.getElementById('markovResults').innerHTML = html;
        }
    </script>
</body>
</html>
