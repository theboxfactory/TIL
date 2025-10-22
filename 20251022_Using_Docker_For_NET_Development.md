# Using Docker for NET x 

docker run -w /app -v $(pwd):/app [image id] [command]

e.g.

docker run -w /app -v $(pwd):/app 5e484330e24d dotnet new console -o test