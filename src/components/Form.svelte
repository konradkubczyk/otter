<script>
  let selectedFile;
  let fileContent;
  let table;
  let reportSpan;
  let clientName;
  let totalAmount;
  let displayedFields = {
    Project: "Project",
    Description: "Description",
    "End date": "Date",
    Duration: "Duration",
  }; // Declare displayedFields as a global variable
  let pricePerHour = localStorage.getItem("pricePerHour") || 0;

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
        )}-${lastEndDate.replaceAll("-", "/")}`;

        // Assign the filtered table data to a variable for display
        table = filteredTableData;

        // Change headers to match displayed fields
        table[0] = table[0].map((header) => displayedFields[header.trim()]);

        // Calculate amounts and insert them into the table
        table.map((row, rowIndex) => {
          if (rowIndex === 0) {
            row.push("Value");
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
            totalAmount += decimalHours * pricePerHour;

            // Round and add amount to row
            row.push(Math.round(decimalHours * pricePerHour * 100) / 100);
          }
        });

        calculateValues();
      };

      reader.readAsText(selectedFile);
    }
  }

  function handlePriceInput(event) {
    pricePerHour = event.target.value;
    calculateValues();

    // Update the table if it exists
    if (table) {
      table = [...table];
    }

    // Save the price per hour to local storage
    localStorage.setItem("pricePerHour", pricePerHour);
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

        row[row.length - 1] =
          Math.round(decimalHours * pricePerHour * 100) / 100;
      }
    });

    // Calculate total amount
    totalAmount = table
      .slice(1)
      .reduce((sum, row) => sum + row[row.length - 1], 0);
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
      placeholder="Price per hour"
      value={pricePerHour}
      on:input={handlePriceInput}
    />
  </form>
</div>

{#if selectedFile && table}
  <div id="report">
    <div class="flex items-center mt-10 mb-2 gap-5">
      <h2 class="text-xl">Summary</h2>
      <div class="divider flex-grow" />
    </div>

    <div class="mb-5">
      <p>Client: <strong>{clientName}</strong></p>
      <p>Report span: <strong>{reportSpan}</strong></p>
      <p>Price per hour: <strong>{pricePerHour}</strong></p>
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
              <td class="px-4 py-2 border-2">{cell}</td>
            {/each}
          </tr>
        {/each}
      </tbody>
      <tfoot>
        <tr>
          <td class="px-4 py-2 border-2" colspan={table[0].length - 1}>
            <strong>Total</strong>
          </td>
          <td class="px-4 py-2 border-2">
            <strong>{Math.round(totalAmount * 100) / 100}</strong>
          </td>
        </tr>
      </tfoot>
    </table>
  </div>
{/if}
