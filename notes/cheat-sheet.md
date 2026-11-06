# Swagger/OpenAPI Cheat Sheet

- Use the Chrome Dev Tools > Console to fetch an API, such as a sanity check like this one that returned a count of the vehicle makes from the NHTSA vpic API
    ```
    fetch("https://vpic.nhtsa.dot.gov/api/vehicles/GetMakesForVehicleType/car?format=json").then(r => r.json()).then(d => console.log(d.Count, d.Message))
    ```
    which responded with:  `195 'Response returned successfully'` 
