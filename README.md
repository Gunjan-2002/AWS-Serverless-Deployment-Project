#Project - 4 : Deploying Serverless Web Application on AWS: S3, API Gateway, Lambda, DynamoDB, and CloudFront

1. First create a DynamoDb table and create partition key as studentid type as string.
2. Then create two Lambda function. One for getting students from database and second one for putting new students to database. Also create new role for DynamoDb so that Lambda function can access database and attach while creating function’s.
    
    **putStudent function**
    
    ```python
    import json
    import boto3
    
    def lambda_handler(event, context):
        # Initialize a DynamoDB resource object for the specified region
        dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
    
        # Select the DynamoDB table named 'studentData'
        table = dynamodb.Table('studentData')
    
        # Scan the table to retrieve all items
        response = table.scan()
        data = response['Items']
    
        # If there are more items to scan, continue scanning until all items are retrieved
        while 'LastEvaluatedKey' in response:
            response = table.scan(ExclusiveStartKey=response['LastEvaluatedKey'])
            data.extend(response['Items'])
    
        # Return the retrieved data
        return data
    ```
    
    **getStudent function**
    
    ```python
    import json
    import boto3
    
    # Create a DynamoDB object using the AWS SDK
    dynamodb = boto3.resource('dynamodb')
    # Use the DynamoDB object to select our table
    table = dynamodb.Table('studentData')
    
    # Define the handler function that the Lambda service will use as an entry point
    def lambda_handler(event, context):
        # Extract values from the event object we got from the Lambda service and store in variables
        student_id = event['studentid']
        name = event['name']
        student_class = event['class']
        age = event['age']
        
        # Write student data to the DynamoDB table and save the response in a variable
        response = table.put_item(
            Item={
                'studentid': student_id,
                'name': name,
                'class': student_class,
                'age': age
            }
        )
        
        # Return a properly formatted JSON object
        return {
            'statusCode': 200,
            'body': json.dumps('Student data saved successfully!')
        }
    ```
    
3. Now create API Gateway having Rest API as a type and then create two methods inside it get and post and attach Lambda function to them.
4. Test API Gateways functionality independently in Test section provided by API Gateway.
5. No create S3 bucket and upload this two files in it.
    
    **index.html**
    
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Student Data</title>
        <style>
            body {
                background-color: #f0f0f0; /* Light gray background */
                color: #333; /* Dark gray text */
                font-family: Arial, sans-serif; /* Use Arial font */
            }
    
            h1 {
                color: #007bff; /* Blue heading text */
            }
    
            .container {
                max-width: 600px; /* Limit width to 600px */
                margin: 0 auto; /* Center the container */
                padding: 20px; /* Add padding */
                background-color: #fff; /* White background */
                border-radius: 10px; /* Rounded corners */
                box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); /* Add shadow */
            }
    
            input[type="text"], input[type="submit"] {
                width: 100%;
                padding: 10px;
                margin: 5px 0;
                box-sizing: border-box;
                border: 1px solid #ccc;
                border-radius: 5px;
            }
    
            input[type="submit"] {
                background-color: #007bff; /* Blue submit button */
                color: #fff; /* White text */
                cursor: pointer; /* Add pointer cursor */
            }
    
            input[type="submit"]:hover {
                background-color: #0056b3; /* Darker blue on hover */
            }
    
            table {
                width: 100%;
                border-collapse: collapse;
            }
    
            th, td {
                padding: 8px;
                text-align: left;
                border-bottom: 1px solid #ddd;
            }
    
            th {
                background-color: #f2f2f2; /* Light gray header background */
            }
        </style>
    </head>
    <body>
        <div class="container">
            <h1>Save and View Student Data</h1>
            <label for="studentid">Student ID:</label><br>
            <input type="text" name="studentid" id="studentid"><br>
            
            <label for="name">Name:</label><br>
            <input type="text" name="name" id="name"><br>
            
            <label for="class">Class:</label><br>
            <input type="text" name="class" id="class"><br>
            
            <label for="age">Age:</label><br>
            <input type="text" name="age" id="age"><br>
            
            <br>
            <input type="submit" id="savestudent" value="Save Student Data">
            <p id="studentSaved"></p>
            
            <br>
            <input type="submit" id="getstudents" value="View all Students">
            <br><br>
            <div id="showStudents">
                <table id="studentTable">
                    <colgroup>
                        <col style="width:20%">
                        <col style="width:20%">
                        <col style="width:20%">
                        <col style="width:20%">
                    </colgroup>
                    <thead>
                        <tr>
                            <th>Student ID</th>
                            <th>Name</th>
                            <th>Class</th>
                            <th>Age</th>
                        </tr>
                    </thead>
                    <tbody>
                        <!-- Student data will be displayed here -->
                    </tbody>
                </table>
            </div>
        </div>
    
        <script src="scripts.js"></script>
        <script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.6.0/jquery.min.js"></script>
    </body>
    </html>
    ```
    
    **Script.js**
    
    Before uploading this file, put API_ENDPOINT value in this file. You will get the endpoint from API Gateway in stages section.
    
    ```jsx
    // Add your API endpoint here
    var API_ENDPOINT = "Sample Endpoint";
    
    // AJAX POST request to save student data
    document.getElementById("savestudent").onclick = function(){
        var inputData = {
            "studentid": $('#studentid').val(),
            "name": $('#name').val(),
            "class": $('#class').val(),
            "age": $('#age').val()
        };
        $.ajax({
            url: API_ENDPOINT,
            type: 'POST',
            data:  JSON.stringify(inputData),
            contentType: 'application/json; charset=utf-8',
            success: function (response) {
                document.getElementById("studentSaved").innerHTML = "Student Data Saved!";
            },
            error: function () {
                alert("Error saving student data.");
            }
        });
    }
    
    // AJAX GET request to retrieve all students
    document.getElementById("getstudents").onclick = function(){  
        $.ajax({
            url: API_ENDPOINT,
            type: 'GET',
            contentType: 'application/json; charset=utf-8',
            success: function (response) {
                $('#studentTable tr').slice(1).remove();
                jQuery.each(response, function(i, data) {          
                    $("#studentTable").append("<tr> \
                        <td>" + data['studentid'] + "</td> \
                        <td>" + data['name'] + "</td> \
                        <td>" + data['class'] + "</td> \
                        <td>" + data['age'] + "</td> \
                        </tr>");
                });
            },
            error: function () {
                alert("Error retrieving student data.");
            }
        });
    }
    ```
    
6. Make S3 bucket as Static Host Enabled and also make the public access enable and add policy to getObjects from S3.
7. Now you can use Static Web Hosting Url to access the website.
8. To enable secure control over the S3 objects as we make bucket public. use Cloudfront in front of S3 bucket and then use Cloudfront Dns name as Website Url.
