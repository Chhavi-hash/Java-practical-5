//  Concurrent Ticket Booking System (self-testing version for Programiz)
const http = require("http");

// In-memory seat data
let seats = {};
for (let i = 1; i <= 10; i++) {
  seats[i] = { status: "available", lockTimer: null };
}

function lockSeat(id) {
  if (!seats[id]) return { statusCode: 400, message: "Invalid seat number" };
  if (seats[id].status === "booked")
    return { statusCode: 400, message: "Seat already booked" };
  if (seats[id].status === "locked")
    return { statusCode: 400, message: "Seat already locked" };

  seats[id].status = "locked";
  // Auto-unlock after 1 minute (for demo: 5 sec)
  seats[id].lockTimer = setTimeout(() => {
    if (seats[id].status === "locked") seats[id].status = "available";
  }, 5000); // use 60000 for 1 minute in real case

  return {
    statusCode: 200,
    message: Seat ${id} locked successfully. Confirm within 1 minute.,
  };
}

function confirmSeat(id) {
  if (!seats[id]) return { statusCode: 400, message: "Invalid seat number" };

  if (seats[id].status === "locked") {
    clearTimeout(seats[id].lockTimer);
    seats[id].status = "booked";
    return { statusCode: 200, message: Seat ${id} booked successfully! };
  } else {
    return {
      statusCode: 400,
      message: "Seat is not locked and cannot be booked",
    };
  }
}

// --- Server setup (for REST API simulation) ---
const PORT = process.env.PORT || 8080;
const server = http.createServer((req, res) => {
  const url = req.url;
  const method = req.method;
  res.setHeader("Content-Type", "application/json");

  // GET /seats
  if (url === "/seats" && method === "GET") {
    res.statusCode = 200;
    res.end(JSON.stringify(seats, null, 2));
  }

  // POST /lock/:id
  else if (url.startsWith("/lock/") && method === "POST") {
    const id = parseInt(url.split("/")[2]);
    const result = lockSeat(id);
    res.statusCode = result.statusCode;
    res.end(JSON.stringify({ message: result.message }));
  }

  // POST /confirm/:id
  else if (url.startsWith("/confirm/") && method === "POST") {
    const id = parseInt(url.split("/")[2]);
    const result = confirmSeat(id);
    res.statusCode = result.statusCode;
    res.end(JSON.stringify({ message: result.message }));
  }

  else {
    res.statusCode = 404;
    res.end(JSON.stringify({ message: "Not Found" }));
  }
});

server.listen(PORT, () => {
  console.log( Server running at http://localhost:${PORT});

  // --- AUTOMATIC TEST SEQUENCE ---
  console.log("\n=== Testing API Simulation ===");

  console.log("\n All seats initially:");
  console.log(JSON.stringify(seats, null, 2));

  console.log("\n Lock seat 5:");
  console.log(lockSeat(5));

  console.log("\n Confirm seat 5:");
  console.log(confirmSeat(5));

  console.log("\n Try confirming seat 2 (not locked):");
  console.log(confirmSeat(2));

  console.log("\n Final seat status:");
  console.log(JSON.stringify(seats, null, 2));

  console.log("\n Test completed. You can also use Postman with the same endpoints!");
});
