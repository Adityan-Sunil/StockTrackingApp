<script>
    import {initializeApp} from 'firebase/app';
    // import { getAnalytics} from 'firebase/analytics';
    import {get, getDatabase, onValue, ref, serverTimestamp, set} from 'firebase/database';
    import { Button, Select, Label, Navbar, NavBrand, NavUl, NavHamburger, Modal } from 'flowbite-svelte';
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
    let stockNames = [];
    let selectedStockName = '';
    let selectedStockData = {};
    let selectedStockDataPointsLabel = '';
    const emptyChartData = {
        Close: [],
        High: [],
        Low: [],
        Open: [],
        Volume: []
    };
    let chartData;
    let defaultModal = false;
    let uploadDate;
    
    clearData();
    initPageData();

    //CSV operations
    function prepareDataForData(csvData) {
        uploadedStockData = csvData;
    }

    function uploadCsvData() {
        for (let index = 1; index < uploadedStockData.length; index++) {
            const row = uploadedStockData[index];
            let obj = {};
            obj['Open'] = row[1];
            obj['High'] = row[2];
            obj['Low'] = row[3];
            obj['Close'] = row[4];
            obj['Volume'] = row[9];
            addDataToDb(row[0], obj,uploadDate);
        }
        defaultModal = false;
    }

    //Firebase CRUD operations
    function addDataToDb(stockName, data, date) {
        set(ref(database, 'stocks/' + stockName + "/" + date), data)
    }
    function readDataFromDb(endpoint) {
        return get(ref(database, endpoint))
    }

    //Stock Data operations
    function getStockData() {
        readDataFromDb('stocks/' + selectedStockName).then((res) => {
            if(res.exists()) {
                selectedStockData = res.val();
                parseStockData();
            }
        });
    }

    //Page functions
    function initPageData() {
        readDataFromDb('stocks').then((res) => {
            if(res.exists()) {
                const elems = Object.keys(res.val());
                stockNames = elems.map((elem) => {return {value: elem, name: elem}});
                selectedStockDataPointsLabel = 'Close';
                selectedStockName = stockNames[0].value;
                getStockData();
            }
        })
        stockDataPointLabels = Object.keys(chartData).map(data => {return {value: data, name: data}})
        const currentDate = new Date();
        const date = currentDate.getDate() < 10 ? '0' + currentDate.getDate() : currentDate.getDate() 
        const month = currentDate.getMonth() < 10 ? '0' + (currentDate.getMonth() + 1) : (currentDate.getMonth() + 1) 
        uploadDate = [date, month, currentDate.getFullYear()].join('-');
    }

    function clearData() {
        chartData = JSON.parse(JSON.stringify(emptyChartData));
    }

    function parseStockData() {
        clearData();
        Object.keys(selectedStockData).forEach(timestamp => {
            Object.keys(selectedStockData[timestamp]).forEach(key => {
                chartData[key].push(parseFloat(selectedStockData[timestamp][key].replace(/,/g, '')));
            })
        })
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
            <Label>Stock
                <Select class="mt-2" items={stockNames} bind:value={selectedStockName} on:change={getStockData} />
            </Label>
            <Label>Data
                <Select class="mt-2" items={stockDataPointLabels} bind:value={selectedStockDataPointsLabel} />
            </Label>
        </div>

        <Chart stockName = {selectedStockName} datapoints={chartData[selectedStockDataPointsLabel]} labels={Object.keys(selectedStockData)}/>
    </div>
</div>

<Modal title="Upload NSE CSV File" bind:open={defaultModal} autoclose>
    <UploadCSV onUpload={(file) =>{prepareDataForData(file)}}/>
    <input type="date" bind:value={uploadDate}>
  <svelte:fragment slot='footer'>
    <Button on:click={uploadCsvData}>Confirm</Button>
    <Button color="alternative">Decline</Button>
  </svelte:fragment>
</Modal>