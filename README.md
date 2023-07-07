# 🦦 Otter

This is a very simple tool for generating summary reports from CSV files containing data about time spent working on projects. It creates PDF files with predefined fields and calculates the total value of work based on customisable hourly rate. It works locally in the browser to protect users' privacy.

## Features

- Simple user interface.
- Total value of work calculation.
- Customizable hourly rate with instant recalculation.
- PDF report generation.
- Private by design.

## Usage

To use this tool on your local machine, follow these steps:

1. Clone the repository and navigate to the project directory.
2. Install the dependencies by running `npm install`.
3. Start the development server with `npm run dev`.
4. Access the tool in your browser at the URL displayed in the terminal.

## How to generate a report

1. Click the "Choose File" button and select a CSV file containing the data you want to generate a report for.
2. Enter the price per hour in the input field.
3. The table with the selected fields from the CSV data will be displayed below.
4. Scroll down to view the summary section, including the client name, report span, and price per hour.
5. Click the "PDF" button to download the report as a PDF file.

> Note: The CSV file must contain the following fields: `Client`, `Project`, `Description`, `End date`, `Duration`.
