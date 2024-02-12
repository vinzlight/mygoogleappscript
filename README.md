code.gs

function doGet() {
  return HtmlService.createHtmlOutputFromFile('index')
      .setTitle('Search and Edit Data');
}

function getData() { 
  var ws = SpreadsheetApp.openById('1Nwuiw4Hi9Rn3FCPPRi2VhVquxkj8iwJ4pSn1XXDdRow').getSheetByName("Database");
  var data = ws.getDataRange().getValues();
  return data;
}

function search(key) {
  var result = [];
  var data = getData();
  for (var i = 0; i < data.length; i++) {
    var id = data[i][0];
    if (id == key) {
      result = data[i];
      break;
    }
  }
  return result;
}

function updateData(updatedFields) {
   Logger.log("Data Received: updatedFields: " + JSON.stringify(updatedFields));  // Log incoming data
  var ws = SpreadsheetApp.openById('1Nwuiw4Hi9Rn3FCPPRi2VhVquxkj8iwJ4pSn1XXDdRow').getSheetByName("Database"); 
  var data = ws.getDataRange().getValues();

  // Find the row corresponding to the updated record
  var key = updatedFields.ID;
  var rowToUpdate = -1;
  for (var i = 0; i < data.length; i++) {
    if (data[i][0] == key) { // Assuming ID is the first column
      rowToUpdate = i;
      break;
      
    }
    
  }

  if (rowToUpdate != -1) {
      Logger.log("Original row data: " + JSON.stringify(data[rowToUpdate])); // Log row before changes
    // Update the data in the corresponding row
    for (var field in updatedFields) {
      var columnIndex = getColumnIndex(field);
       Logger.log("Updating field: " + field + ", columnIndex: " + columnIndex + ", Value: " +  updatedFields[field]); // Add logging
      if (columnIndex != -1) {
        data[rowToUpdate][columnIndex] = updatedFields[field];
      }
    }
 Logger.log("Updated row data: " + JSON.stringify(data[rowToUpdate])); // Log row after changes
    // Write the updated data back to the sheet
    ws.getRange(1, 1, data.length, data[0].length).setValues(data);
    return "Data updated successfully.";
  } else {
    return "Record with ID " + key + " not found.";
  }
}

function getColumnIndex(fieldName) {
  // ... (rest of your function) ... 

  for (var i = 0; i < headerRow.length; i++) {
    if (headerRow[i].toLowerCase() == fieldName.toLowerCase()) { // Case-insensitive
      return i;
    }
  }
  return -1; 
}

----------------------

index.html

<!DOCTYPE html>
<html>
<head>
    <base target="_top">
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
    
    <style>
        /* Custom CSS if needed */
        .error {
            color: red;
        }
    </style>
</head>
<body>
    <div class="container mt-5">
        <div class="row">
            <div class="col-md-6 mx-auto">
                <div class="card">
                    <div class="card-body">
                        <h5 class="card-title">Search ID</h5>
                        <div class="form-group">
                            <input type="text" id="key" class="form-control" placeholder="Enter ID">
                        </div>
                        <button type="button" class="btn btn-primary btn-block" onclick="search()">Search</button>
                        <span id="form-msg" class="error mt-3"></span>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Modal -->
    <div class="modal fade" id="resultModal" tabindex="-1" role="dialog" aria-labelledby="resultModalLabel" aria-hidden="true">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="resultModalLabel">Edit Person Information</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <div class="modal-body" id="resultModalBody">
                    <!-- Data from sheet will be displayed here -->
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                    <button type="button" class="btn btn-primary" onclick="saveChanges()">Save changes</button>
                </div>
            </div>
        </div>
    </div>
    
    <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@popperjs/core@2.5.4/dist/umd/popper.min.js"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
    
    <script>
        var formMsg = document.getElementById("form-msg");

        function search() {
            var key = document.getElementById("key").value;
            google.script.run
                .withSuccessHandler(function(result) {
                    if (result.length == 0) {
                        formMsg.innerHTML = "No result found for this ID.";
                    } else {
                        displayModal(result);
                    }
                })
                .withFailureHandler(function(error) {
                    formMsg.innerHTML = error;
                })
                .search(key);
        }

        function displayModal(result) {
            var modalBody = document.getElementById("resultModalBody");
            modalBody.innerHTML = ""; // Clear previous content
            var fields = ["ID", "First Name", "Last Name", "Gender", "Email", "Phone", "Address", "City", "Country"];
            for (var i = 0; i < fields.length; i++) {
                var row = '<div class="form-group row">';
                row += '<label for="' + fields[i].replace(/\s/g, '') + '" class="col-sm-4 col-form-label">' + fields[i] + '</label>';
                row += '<div class="col-sm-8">';
                row += '<input type="text" class="form-control" id="' + fields[i].replace(/\s/g, '') + '" value="' + result[i] + '">';
                row += '</div>';
                row += '</div>';
                modalBody.innerHTML += row;
            }
            $('#resultModal').modal('show');
        }

       function saveChanges() {
    var updatedFields = {};
    var fields = ["ID", "First Name", "Last Name", "Gender", "Email", "Phone", "Address", "City", "Country"];
    for (var i = 0; i < fields.length; i++) {
        var fieldName = fields[i].replace(/\s/g, '');
        updatedFields[fieldName] = document.getElementById(fieldName).value;
            console.log("Data to send to updateData: ", updatedFields); // Console log updatedFields

    }
    google.script.run
        .withSuccessHandler(function() {
            $('#resultModal').modal('hide');
            formMsg.innerHTML = "Changes saved successfully.";
        })
        .withFailureHandler(function(error) {
            formMsg.innerHTML = error;
        })
        .updateData(updatedFields); // Corrected function name
}
    </script>
</body>
</html>






