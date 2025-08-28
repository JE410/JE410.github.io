<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Test Org Chart</title>
<!-- OrgChartJS CSS & JS -->
<script src="https://cdn.balkan.app/orgchart.js"></script>
<style>
   body {
     font-family: Arial, sans-serif;
     padding: 10px;
   }
   #orgChartContainer {
     width: 100%;
     height: 600px;
   }
</style>
</head>
<body>
<h2>Organization Chart (Test)</h2>
<div id="tree"></div>
<script>

    let chart = new OrgChart("#tree", {
        // options
        nodeBinding: {
            field_0: "name"
        },
        nodes: [
            { id: 1, name: "Amber McKenzie" },
            { id: 2, pid: 1, name: "Ava Field" },
            { id: 3, pid: 1, name: "Peter Stevens" }
        ]  
    });
 
</script> 
</body>
</html>
