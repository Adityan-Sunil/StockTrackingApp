<script>
    import {initializeApp} from 'firebase/app';
    // import { getAnalytics} from 'firebase/analytics';
    import {get, getDatabase, onValue, ref, serverTimestamp, set} from 'firebase/database';
    import { Button, Select, Label, Navbar, NavBrand, NavUl, NavHamburger, Modal, Dropdown, Checkbox, Chevron, Search } from 'flowbite-svelte';
    import UploadCSV from 'upload-csv-svelte';
	import Chart from '../components/Chart.svelte';
 
    const firebaseConfig = {
        apiKey: "AIzaSyDk8L4wRsze8Xk59GKvHtn_XwP66QOLfgQ",
        authDomain: "stocktracking-c66a0.firebaseapp.com",
        projectId: "stocktracking-c66a0",
        storageBucket: "stocktracking-c66a0.appspot.com",
        messagingSenderId: "299896818013",
        appId: "1:299896818013:web:ba06c5db4af8e6217cc0b4",
        measurementId: "G-3MP8KVWKT4",
        databaseURL: 'https://stocktracking-c66a0-default-rtdb.asia-southeast1.firebasedatabase.app'
    };

    const app = initializeApp(firebaseConfig);
    // const analytics = getAnalytics(app);
    const database = getDatabase(app);


    let uploadedStockData;
    let stockDataPointLabels = [];
    let displayStockNameList = [];
    let selectedStockData = {};
    let selectedStockDataPointsLabel = '';
    let stockNameSelectionList = []
    $: stockNameSelectedList = []
    let filteredDisplayNameList =[];
    let filteredStockNameSelectionList = [];
    let stockNameInput;
    let chartData;
    let defaultModal = false;
    let uploadDate;
    let displayStockName

    clearData();
    initPageData();

    $: if(!defaultModal) {
        uploadDate = '';
        stockNameInput = '';
        stockNameSelectionList = [];
    }

    //CSV operations
    function prepareDataForData(csvData) {
        uploadedStockData = csvData;
        stockNameSelectionList = uploadedStockData.map(stockData => stockData[0]);
        stockNameSelectionList.shift();
    }

    function uploadCsvData() {
        if(stockNameSelectedList.length <= 0) {
            alert("No stock names selected");
            return;
        }
        if(!uploadDate) {
            alert("No date selected");
            return;
        }
        saveStockNameList();
        stockNameSelectedList.forEach(stockName =>{
            let index = stockNameSelectionList.indexOf(stockName) + 1;
            const row = uploadedStockData[index];
            let obj = {};
            obj['Open'] = row[1];
            obj['High'] = row[2];
            obj['Low'] = row[3];
            obj['Close'] = row[4];
            obj['Volume'] = row[9];
            addDataToDb(row[0], obj,uploadDate);
        })
        defaultModal = false;
    }

    //Firebase CRUD operations
    function saveStockNameList() {
        set(ref(database, 'savedStocks'), stockNameSelectedList.join(','));
    }
    function addDataToDb(stockName, data, date) {
        set(ref(database, 'stocks/' + stockName + "/" + date), data)
    }
    function readDataFromDb(endpoint) {
        return get(ref(database, endpoint))
    }

    //Stock Data operations
    function getStockData(stockName) {
        readDataFromDb('stocks/' + stockName).then((res) => {
            if(res.exists()) {
                selectedStockData[stockName] = res.val();
                parseStockData();
            }
        });
    }

    //Page functions
    function initPageData() {
        readDataFromDb('savedStocks').then((res) => {
            if(res.exists() && res.val() !== "") {
                stockNameSelectedList = res.val().split(',');
                stockNameSelectedList.forEach(stockName => getStockData(stockName));
                displayStockNameList = Array.from(stockNameSelectedList);
                filteredDisplayNameList = filterInputValueOnList(displayStockNameList, undefined, true);
            } else {
                defaultModal = true;
            }
        });     

        const savedStockRef = ref(database,'savedStocks');
        const stockRef = ref(database, 'stocks/');

        onValue(savedStockRef, (data) => {
            if(data.exists() && data.val() !== "") {
                stockNameSelectedList = data.val().split(',');
                displayStockNameList = Array.from(stockNameSelectedList);
                filteredDisplayNameList = filterInputValueOnList(displayStockNameList, undefined, true);
            }
        });
        
        onValue(stockRef, (data) => {
            if(data.exists()) {
                const stocks = Object.keys(data.val());
                stocks.forEach(stock => selectedStockData[stock] = data.val()[stock]);
                parseStockData()
            }
        })

    }

    function clearData() {
        chartData = []
    }

    function filterInputValueOnList(listToBeFiltered, filterString, returnOriginalArray = false) {
        let filteredList = [];
        if(filterString) {
            filteredList = listToBeFiltered.filter(name => name.toLowerCase().startsWith(filterString.toLowerCase()))
        } else if(returnOriginalArray){
            filteredList = Array.from(listToBeFiltered);
        }
        return filteredList;
    }
    
    function removeNameFromStockList(stockNameList, stockName) {
        const index = stockNameList.indexOf(stockName);
        let tempArr = Array.from(stockNameList);
        tempArr.splice(index, 1);
        return tempArr
    }

    function addNameToStockList(stockNameList, name) {
        let tempArr = Array.from(stockNameList);
        tempArr.push(name);
        return tempArr
    }

    function addNameToSelectedStockList(name) {
        stockNameSelectedList = addNameToStockList(stockNameSelectedList, name);
        filteredStockNameSelectionList = [];
        stockNameInput = name;
    }

    function dropDownValueChange(checkedValue, stockName) {
        if(checkedValue) {
           displayStockNameList = addNameToStockList(displayStockNameList,stockName);
        } else {
           displayStockNameList = removeNameFromStockList(displayStockNameList,stockName);
        }
    }

    function parseStockData() {
        clearData();
        Object.keys(selectedStockData).forEach(stockName => {
            const dataObj = {};
            const stockData = selectedStockData[stockName]
            Object.keys(stockData).forEach(timestamp => {
                Object.keys(stockData[timestamp]).forEach(key => {
                    if(!dataObj[key]) {
                        dataObj[key] = [];
                    }
                    dataObj[key].push(parseFloat(stockData[timestamp][key].replace(/,/g, '')));
                })
            })
            chartData[stockName] = dataObj;
        })
        stockDataPointLabels = Object.keys(chartData[(Object.keys(chartData).at(0))]).map(data => {return {value: data, name: data}});
        selectedStockDataPointsLabel = stockDataPointLabels[0].value;
    }

</script>

<div class="relative px-8">
    <Navbar
      let:hidden
      let:toggle
    >
        <NavBrand href="/">
        <img src="https://flowbite.com/docs/images/logo.svg" class="mr-3 h-6 sm:h-9" alt="Flowbite Logo"/>
        <span class="self-center whitespace-nowrap text-xl font-semibold dark:text-white">Stock Tracking</span>
        </NavBrand>
        <NavHamburger on:click={toggle} />
        <NavUl {hidden}>
        <Button on:click={() => defaultModal = true}>Upload File</Button>
        </NavUl>
    </Navbar>
    <div class="mx-auto container py-2">
        <div class="flex justify-between">
            <Label>
                <Select items={stockDataPointLabels} bind:value={selectedStockDataPointsLabel} />
            </Label>
            <div>
                <Search style="width: 800px" size="md" bind:value={displayStockName} on:input={() => displayStockNameList = filterInputValueOnList(stockNameSelectedList, displayStockName, true)} />
                <div class="flex flex-wrap" style="width: 800px">
                    {#each displayStockNameList as stockName}
                    <span id="badge-dismiss-default" class="inline-flex items-start px-2 py-1 mr-2 mt-1 text-sm font-medium text-blue-800 bg-blue-100 rounded dark:bg-blue-900 dark:text-blue-300">
                        {stockName}
                        <button type="button" class="inline-flex items-start p-0.5 ml-2 text-sm text-blue-400 bg-transparent rounded-sm hover:bg-blue-200 hover:text-blue-900 dark:hover:bg-blue-800 dark:hover:text-blue-300" data-dismiss-target="#badge-dismiss-default" on:click={dropDownValueChange(false,stockName)} aria-label="Remove">
                            <svg aria-hidden="true" class="w-3.5 h-3.5" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd"></path></svg>
                            <span class="sr-only">Remove badge</span>
                        </button>
                    </span>
                {/each}
                </div>
            </div>
            <div>
                <Button><Chevron>Selected Stocks</Chevron></Button>
                <Dropdown class="overflow-y-auto px-3 pb-3 text-sm h-44">
                    <div slot="header" class="p-3">
                        <Search size="md" bind:value={displayStockName} on:input={() => filteredDisplayNameList = filterInputValueOnList(stockNameSelectedList, displayStockName, true)} />
                    </div>
                    {#each filteredDisplayNameList as stockName}
                    <li class="rounded p-3 hover:bg-gray-100 dark:hover:bg-gray-600">
                        <Checkbox checked={displayStockNameList.indexOf(stockName) > -1} on:change={($event) => dropDownValueChange($event.target.checked, stockName)}>{stockName}</Checkbox>
                    </li>
                    {/each}
                </Dropdown>
            </div>
        </div>

        <div class="chart-grid grid grid-cols-2 gap-4 mt-10">
            {#each Object.keys(chartData) as stockName}
                {#if displayStockNameList.indexOf(stockName) > -1}
                <div class='chart-grid-element'>
                    <h2>{stockName}</h2>
                    <div style="height: 510px; padding: 1%">
                        <Chart {stockName} datapoints={chartData[stockName]} dataType={selectedStockDataPointsLabel} labels = {Object.keys(selectedStockData[stockName])}/>
                    </div>
                </div>
                {/if}
            {/each}
        </div>
    </div>
</div>

<Modal title="Upload NSE CSV File" bind:open={defaultModal}>
    <div>
        <UploadCSV onUpload={(file) =>{prepareDataForData(file)}}/>
        <input type="date" bind:value={uploadDate}>
    </div>
    {#if stockNameSelectionList.length > 0}
        <input class="stockNameSelectionInput" type="text" bind:value={stockNameInput} on:input={() =>{filteredStockNameSelectionList = filterInputValueOnList(stockNameSelectionList,stockNameInput)}} placeholder="Select Stock Name">
        {#if filteredStockNameSelectionList.length > 0}
            <ul class="filterStockList" style="">
                {#each filteredStockNameSelectionList as filteredName}
                    <li class="input-auto-complete">
                        <button on:click={addNameToSelectedStockList(filteredName)}>{filteredName}</button>
                    </li>
                {/each}
            </ul>
        {/if}
        <div>
            {#each stockNameSelectedList as stockName}
                <span id="badge-dismiss-default" class="inline-flex items-center px-2 py-1 mr-2 text-sm font-medium text-blue-800 bg-blue-100 rounded dark:bg-blue-900 dark:text-blue-300">
                    {stockName}
                    <button type="button" class="inline-flex items-center p-0.5 ml-2 text-sm text-blue-400 bg-transparent rounded-sm hover:bg-blue-200 hover:text-blue-900 dark:hover:bg-blue-800 dark:hover:text-blue-300" data-dismiss-target="#badge-dismiss-default" on:click={() => stockNameSelectedList = removeNameFromStockList(stockNameSelectedList,stockName)} aria-label="Remove">
                        <svg aria-hidden="true" class="w-3.5 h-3.5" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd"></path></svg>
                        <span class="sr-only">Remove badge</span>
                    </button>
                </span>
            {/each}
        </div>
    {/if}
  <svelte:fragment slot='footer'>
    <Button on:click={uploadCsvData}>Confirm</Button>
    <Button color="alternative" on:click={() => {defaultModal = false;}}>Decline</Button>
  </svelte:fragment>
</Modal>

<style>
    ul.filterStockList {
        position: absolute;
        width:300px;
        max-height:100px; 
        margin:0 !important;
        overflow-y: scroll;
        padding: 0;
    }
    li.input-auto-complete {
        border: 1px solid black;
        border-top: 0;
        padding: 1%;
        background: white;
    }
    li.input-auto-complete>button {
        width: 100%;
    }
    input.stockNameSelectionInput {
        width:300px;
        position: relative
    }
    h2{
        background: rgb(176, 219, 255);
        padding: 1%;
    }
    .chart-grid-element {
        border: 1px solid rgb(213, 235, 253);
        box-sizing: border-box;
    }

</style>