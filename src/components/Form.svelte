<script>
  import pdfMake from "pdfmake/build/pdfmake";
  import htmlToPdfmake from "html-to-pdfmake";

  pdfMake.fonts = {
    // download default Roboto font from cdnjs.com
    Roboto: {
      normal:
        "https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.1.66/fonts/Roboto/Roboto-Regular.ttf",
      bold: "https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.1.66/fonts/Roboto/Roboto-Medium.ttf",
      italics:
        "https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.1.66/fonts/Roboto/Roboto-Italic.ttf",
      bolditalics:
        "https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.1.66/fonts/Roboto/Roboto-MediumItalic.ttf",
    },
  };

  let selectedFile;
  let fileContent;
  let table;
  let reportSpan;
  let clientName;
  let totalAmount;
  let displayedFields = {
    Project: "Projekt",
    Description: "Opis",
    "End date": "Data",
    Duration: "Czas",
  }; // Declare displayedFields as a global variable
  let hourlyRate = localStorage.getItem("hourlyRate") || 0;

  function handleFileInput(event) {
    totalAmount = 0;
    selectedFile = event.target.files[0];

    if (selectedFile) {
      const reader = new FileReader();

      reader.onload = () => {
        fileContent = reader.result;
        const rows = fileContent.split("\n");
        const tableData = rows.map((row) => row.split(","));

        // Remove empty rows
        tableData.forEach((row, rowIndex) => {
          if (row.length === 1 && row[0] === "") {
            tableData.splice(rowIndex, 1);
          }
        });

        // Get client name
        clientName = tableData[1][tableData[0].indexOf("Client")];

        // Ensure that there is only one client in the file
        tableData.forEach((row, rowIndex) => {
          if (
            rowIndex !== 0 &&
            row[tableData[0].indexOf("Client")] !== clientName
          ) {
            alert("Error: Multiple clients found in the file");
            selectedFile = null;
            return;
          }
        });

        // Filter the table data based on displayed fields
        const filteredTableData = tableData.map((row) =>
          row.filter((_, columnIndex) =>
            Object.keys(displayedFields).includes(
              tableData[0][columnIndex].trim()
            )
          )
        );

        // Get first and last End date
        const firstEndDate =
          filteredTableData[1][
            Object.keys(displayedFields).indexOf("End date")
          ];
        const lastEndDate =
          filteredTableData[filteredTableData.length - 1][
            Object.keys(displayedFields).indexOf("End date")
          ];

        // Generate report range based on first and last End date
        reportSpan = `${firstEndDate.replaceAll(
          "-",
          "/"
        )} - ${lastEndDate.replaceAll("-", "/")}`;

        // Assign the filtered table data to a variable for display
        table = filteredTableData;

        // Change headers to match displayed fields
        table[0] = table[0].map((header) => displayedFields[header.trim()]);

        // Calculate amounts and insert them into the table
        table.map((row, rowIndex) => {
          if (rowIndex === 0) {
            row.push("Wartość");
          } else {
            const duration =
              row[Object.keys(displayedFields).indexOf("Duration")];

            // Convert hh:mm:ss to decimal hours
            const hours = duration.split(":")[0];
            const minutes = duration.split(":")[1];
            const seconds = duration.split(":")[2];
            const decimalHours =
              Number(hours) + Number(minutes) / 60 + Number(seconds) / 3600;

            // Increase total amount
            totalAmount += decimalHours * hourlyRate;

            // Round and add amount to row
            row.push(Math.round(decimalHours * hourlyRate * 100) / 100);
          }
        });

        calculateValues();
      };

      reader.readAsText(selectedFile);
    }
  }

  function handleHourlyRateInput(event) {
    hourlyRate = event.target.value;
    calculateValues();

    // Update the table if it exists
    if (table) {
      table = [...table];
    }

    // Save the hourly rate to local storage
    localStorage.setItem("hourlyRate", hourlyRate);
  }

  function calculateValues() {
    // Check if table exists
    if (!table) {
      return;
    }

    // Calculate amounts and insert them into the table
    table.map((row, rowIndex) => {
      if (rowIndex !== 0) {
        const duration = row[Object.keys(displayedFields).indexOf("Duration")];
        const hours = duration.split(":")[0];
        const minutes = duration.split(":")[1];
        const seconds = duration.split(":")[2];
        const decimalHours =
          Number(hours) + Number(minutes) / 60 + Number(seconds) / 3600;

        row[row.length - 1] = Math.round(decimalHours * hourlyRate * 100) / 100;
      }
    });

    // Calculate total amount
    totalAmount = table
      .slice(1)
      .reduce((sum, row) => sum + row[row.length - 1], 0);
  }

  function generatePDF() {
    // Get #report element
    const report = document.getElementById("report");

    // Convert #report element to pdfmake-compatible object
    const reportObject = htmlToPdfmake(report.innerHTML);

    // Add title to reportObject
    reportObject.unshift({
      text: "Raport",
      style: "header",
      margin: [0, 0, 0, 10],
      fontSize: 20,
    });

    // Create PDF document with name
    pdfMake
      .createPdf({
        content: [reportObject],
      })
      .download(`${clientName} - ${reportSpan.replaceAll("/", ".")}.pdf`);
  }
</script>

<div>
  <form class="flex gap-3">
    <input
      type="file"
      class="file-input w-full outline outline-gray-300 focus:outline-gray-500 outline-2 outline-offset-2"
      accept=".csv"
      on:change={handleFileInput}
    />
    <input
      type="number"
      class="input w-full max-w-xs outline outline-gray-300 focus:outline-gray-500 outline-2 outline-offset-2"
      placeholder="Hourly rate"
      value={hourlyRate}
      on:input={handleHourlyRateInput}
    />
  </form>
</div>

{#if selectedFile && table}
  <div class="flex items-center mt-10 mb-2 gap-5">
    <h2 class="text-xl">Raport</h2>
    <div class="divider flex-grow" />
    <button class="btn" on:click={generatePDF}>PDF</button>
  </div>

  <div id="report">
    <div class="mb-5">
      <p>
        Klient: <strong>{clientName}</strong>
        <br />
        Zakres: <strong>{reportSpan}</strong>
        <br />
        Stawka: <strong>{hourlyRate}</strong>
      </p>
    </div>

    <table class="border-collapse w-full">
      <thead>
        <tr>
          {#each table[0] as header}
            <th class="px-4 py-2 border-2">{header}</th>
          {/each}
        </tr>
      </thead>
      <tbody>
        {#each table.slice(1) as row}
          <tr>
            {#each row as cell, columnIndex}
              <td class="px-4 py-2 border-2">
                {#if columnIndex === row.length - 1}
                  {parseFloat(cell).toFixed(2)}
                {:else}
                  {cell}
                {/if}
              </td>
            {/each}
          </tr>
        {/each}
      </tbody>
      <tfoot>
        <tr>
          <td class="px-4 py-2 border-2" colspan={table[0].length - 1}>
            <strong>Suma</strong>
          </td>
          <td class="px-4 py-2 border-2">
            <strong>{(Math.round(totalAmount * 100) / 100).toFixed(2)}</strong>
          </td>
        </tr>
      </tfoot>
    </table>
  </div>
{/if}
