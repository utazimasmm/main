import { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Table, TableHeader, TableRow, TableHead, TableBody, TableCell } from "@/components/ui/table";

const API_BASE = "https://UtazimaSMM.com";

export default function SMMPanel() {
  const [balance, setBalance] = useState(0);
  const [orders, setOrders] = useState([]);
  const [service, setService] = useState("Instagram Likes");
  const [quantity, setQuantity] = useState(100);
  const [token, setToken] = useState(null);
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [isSignUp, setIsSignUp] = useState(false);

  const toggleAuthMode = () => setIsSignUp(!isSignUp);

  const signUp = () => {
    fetch(`${API_BASE}/signup`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ username, password })
    })
      .then(res => res.json())
      .then(data => {
        if (data.success) {
          alert("Account created! You can now log in.");
          setIsSignUp(false);
        } else {
          alert("Sign-up failed: " + data.message);
        }
      });
  };

  const login = () => {
    fetch(`${API_BASE}/login`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ username, password })
    })
      .then(res => res.json())
      .then(data => {
        if (data.success) {
          setToken(data.token);
          setIsAuthenticated(true);
          fetchData(data.token);
        } else {
          alert("Invalid credentials");
        }
      });
  };

  const fetchData = (authToken) => {
    fetch(`${API_BASE}/balance`, {
      headers: { Authorization: authToken }
    })
      .then(res => res.json())
      .then(data => setBalance(data.balance));
    
    fetch(`${API_BASE}/orders`, {
      headers: { Authorization: authToken }
    })
      .then(res => res.json())
      .then(data => setOrders(data.orders));
  };

  const placeOrder = () => {
    fetch(`${API_BASE}/order`, {
      method: "POST",
      headers: { "Content-Type": "application/json", Authorization: token },
      body: JSON.stringify({ service, quantity })
    })
      .then(res => res.json())
      .then(data => {
        if (data.success) {
          setOrders([...orders, data.order]);
          setBalance(data.balance);
        } else {
          alert("Insufficient balance!");
        }
      });
  };

  if (!isAuthenticated) {
    return (
      <div className="p-6 space-y-6">
        <h1 className="text-2xl font-bold">{isSignUp ? "Sign Up" : "Login"}</h1>
        <Input type="text" value={username} onChange={(e) => setUsername(e.target.value)} placeholder="Email" />
        <Input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" />
        {isSignUp ? (
          <Button onClick={signUp}>Sign Up</Button>
        ) : (
          <Button onClick={login}>Login</Button>
        )}
        <p className="text-sm cursor-pointer text-blue-500" onClick={toggleAuthMode}>
          {isSignUp ? "Already have an account? Login" : "Don't have an account? Sign up"}
        </p>
      </div>
    );
  }

  return (
    <div className="p-6 space-y-6">
      <h1 className="text-2xl font-bold">SMM Panel Dashboard</h1>
      
      <Card>
        <CardContent className="p-4">
          <p className="text-lg font-semibold">Balance: ${balance.toFixed(2)}</p>
        </CardContent>
      </Card>

      <Card>
        <CardContent className="p-4 space-y-4">
          <h2 className="text-lg font-semibold">Place New Order</h2>
          <Input type="text" value={service} onChange={(e) => setService(e.target.value)} placeholder="Service Name" />
          <Input type="number" value={quantity} onChange={(e) => setQuantity(Number(e.target.value))} placeholder="Quantity" />
          <Button onClick={placeOrder}>Submit Order</Button>
        </CardContent>
      </Card>

      <Card>
        <CardContent className="p-4">
          <h2 className="text-lg font-semibold">Orders</h2>
          <Table>
            <TableHeader>
              <TableRow>
                <TableHead>ID</TableHead>
                <TableHead>Service</TableHead>
                <TableHead>Quantity</TableHead>
                <TableHead>Status</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              {orders.map((order) => (
                <TableRow key={order.id}>
                  <TableCell>{order.id}</TableCell>
                  <TableCell>{order.service}</TableCell>
                  <TableCell>{order.quantity}</TableCell>
                  <TableCell>{order.status}</TableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        </CardContent>
      </Card>
    </div>
  );
}
