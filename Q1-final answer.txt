// Helper functions for Base36 encoding and decoding
function encodeBase36(num, length) {
  // Encodes a number into Base36 and pads it to the specified length
  return num.toString(36).padStart(length, "0").toUpperCase();
}

function decodeBase36(str) {
  // Decodes a Base36 string back into a number
  return parseInt(str, 36);
}

// Function to calculate the current day of the year
function getDayOfYear(date) {
  // Calculates the day of the year based on the input date
  const start = new Date(date.getFullYear(), 0, 0);
  const diff =
    date -
    start +
    (start.getTimezoneOffset() - date.getTimezoneOffset()) * 60 * 1000;
  return Math.floor(diff / (1000 * 60 * 60 * 24));
}

// Generates a short code
function generateShortCode(storeId, transactionId) {
  // Problem in the original code: It returned a hardcoded value ("ABCDEFGHI") instead of a dynamic short code
  // Fix: Use Base36 encoding to dynamically generate a short code

  const today = new Date();
  const dayOfYear = getDayOfYear(today);

  // Encode each part into Base36 for compact representation
  const storeCode = encodeBase36(storeId, 2); // Limit: 2 characters
  const dayCode = encodeBase36(dayOfYear, 3); // Limit: 3 characters
  const transactionCode = encodeBase36(transactionId, 3); // Limit: 3 characters

  // Combine parts to form a short code
  return storeCode + dayCode + transactionCode;
}

// Decodes a short code back to original values
function decodeShortCode(shortCode) {
  // Problem in the original code: It always returned default values (storeId: 0, transactionId: 0, shopDate: new Date())
  // Fix: Decode each part of the short code back into storeId, transactionId, and shopDate

  const storeCode = shortCode.slice(0, 2); // Extract store part
  const dayCode = shortCode.slice(2, 5); // Extract day part
  const transactionCode = shortCode.slice(5, 8); // Extract transaction part

  // Decode Base36 parts into numbers
  const storeId = decodeBase36(storeCode);
  const dayOfYear = decodeBase36(dayCode);
  const transactionId = decodeBase36(transactionCode);

  // Calculate the shop date from the day of the year
  const today = new Date();
  const shopDate = new Date(today.getFullYear(), 0, dayOfYear);

  return {
    storeId: storeId,
    shopDate: shopDate,
    transactionId: transactionId,
  };
}

function RunTests() {
  var storeIds = [175, 42, 0, 9];
  var transactionIds = [9675, 23, 123, 7];

  storeIds.forEach(function (storeId) {
    transactionIds.forEach(function (transactionId) {
      var shortCode = generateShortCode(storeId, transactionId);
      var decodeResult = decodeShortCode(shortCode);

      // Append results to DOM (this area is not related to the core logic)
      $("#test-results").append(
        "<div>" + storeId + " - " + transactionId + ": " + shortCode + "</div>"
      );

      // Run tests
      AddTestResult("Length <= 9", shortCode.length <= 9); // Check short code length
      AddTestResult("Is String", typeof shortCode === "string"); // Check if it's a string
      AddTestResult("Is Today", IsToday(decodeResult.shopDate)); // Check if shopDate matches today's date
      AddTestResult("StoreId", storeId === decodeResult.storeId); // Check if storeId matches
      AddTestResult("TransId", transactionId === decodeResult.transactionId); // Check if transactionId matches
    });
  });
}

function IsToday(inputDate) {
  // Compares inputDate with today's date, ignoring time
  var todaysDate = new Date();
  return inputDate.setHours(0, 0, 0, 0) === todaysDate.setHours(0, 0, 0, 0);
}

function AddTestResult(testName, testResult) {
  // Appends test results to the DOM
  var div = $("#test-results").append(
    "<div class='" +
      (testResult ? "pass" : "fail") +
      "'><span class='tname'>- " +
      testName +
      "</span><span class='tresult'>" +
      testResult +
      "</span></div>"
  );
}
