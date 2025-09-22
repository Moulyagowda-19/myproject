"use client";

import React, { useState, useRef } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import {
Select,
SelectContent,
SelectItem,
SelectTrigger,
SelectValue,
} from "@/components/ui/select";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Badge } from "@/components/ui/badge";
import {
BarChart,
Bar,
LineChart,
Line,
XAxis,
YAxis,
CartesianGrid,
Tooltip,
Legend,
ResponsiveContainer,
Cell,
} from "recharts";
import {
Upload,
Trash2,
GripVertical,
Plus,
Table,
BarChart3,
TrendingUp,
} from "lucide-react";
import { motion } from "framer-motion";

// Types
type DataItem = Record\<string, string | number>;
type VisualizationType = "bar" | "line" | "table";
type ChartConfig = {
id: string;
type: VisualizationType;
title: string;
xAxis: string;
yAxis: string;
data: DataItem\[];
};

const DashboardCreator: React.FC = () => {
// State
const \[data, setData] = useState\<DataItem\[]>(\[]);
const \[columns, setColumns] = useState\<string\[]>(\[]);
const \[charts, setCharts] = useState\<ChartConfig\[]>(\[]);
const \[draggingId, setDraggingId] = useState\<string | null>(null);
const fileInputRef = useRef<HTMLInputElement>(null);

// Handle file upload
const handleFileUpload = (event: React.ChangeEvent<HTMLInputElement>) => {
const file = event.target.files?.\[0];
if (!file) return;

```
// ✅ reset input so same file can reupload
event.target.value = "";

const reader = new FileReader();
reader.onload = (e) => {
  try {
    const content = e.target?.result as string;
    if (file.name.endsWith(".csv")) {
      parseCSV(content);
    } else if (file.name.endsWith(".json")) {
      parseJSON(content);
    }
  } catch (error) {
    console.error("Error parsing file:", error);
    alert("Error parsing file. Please check the format.");
  }
};
reader.readAsText(file);
```

};

// Parse CSV data
const parseCSV = (content: string) => {
const lines = content.split("\n").filter((line) => line.trim() !== "");
if (lines.length < 2) return;

```
const headers = lines[0].split(",").map((h) => h.trim());
const parsedData = lines.slice(1).map((line) => {
  const values = line.split(",");
  const item: DataItem = {};
  headers.forEach((header, index) => {
    const value = values[index]?.trim() || "";
    // Try to convert to number if possible
    item[header] = isNaN(Number(value)) ? value : Number(value);
  });
  return item;
});

setColumns(headers);
setData(parsedData);
```

};

// Parse JSON data
const parseJSON = (content: string) => {
const parsedData = JSON.parse(content);
if (!Array.isArray(parsedData) || parsedData.length === 0) {
throw new Error("Invalid JSON format");
}

```
const headers = Object.keys(parsedData[0]);
setColumns(headers);
setData(parsedData);
```

};

// Add a new chart
const addChart = (type: VisualizationType) => {
if (data.length === 0) {
alert("Please upload data first");
return;
}

```
const newChart: ChartConfig = {
  id: `chart-${Date.now()}`,
  type,
  title: `${type.charAt(0).toUpperCase() + type.slice(1)} Chart`,
  xAxis: columns[0] || "",
  yAxis: columns[1] || columns[0] || "", // ✅ fallback
  data,
};

setCharts([...charts, newChart]);
```

};

// Update chart configuration
const updateChart = (id: string, updates: Partial<ChartConfig>) => {
setCharts(
charts.map((chart) => (chart.id === id ? { ...chart, ...updates } : chart))
);
};

// Remove chart
const removeChart = (id: string) => {
setCharts(charts.filter((chart) => chart.id !== id));
};

// Drag and drop handlers
const handleDragStart = (id: string) => {
setDraggingId(id);
};

const handleDragOver = (e: React.DragEvent) => {
e.preventDefault();
};

const handleDrop = (targetId: string) => {
if (!draggingId || draggingId === targetId) return;

```
const draggingIndex = charts.findIndex((c) => c.id === draggingId);
const targetIndex = charts.findIndex((c) => c.id === targetId);

if (draggingIndex === targetIndex) return; // ✅ prevent no-op drag

const newCharts = [...charts];
const [removed] = newCharts.splice(draggingIndex, 1);
newCharts.splice(targetIndex, 0, removed);

setCharts(newCharts);
setDraggingId(null);
```

};

// Render visualization based on type
const renderVisualization = (chart: ChartConfig) => {
switch (chart.type) {
case "bar":
return ( <ResponsiveContainer width="100%" height={300}> <BarChart data={chart.data}> <CartesianGrid strokeDasharray="3 3" stroke="#e5e7eb" /> <XAxis dataKey={chart.xAxis} stroke="#6b7280" /> <YAxis stroke="#6b7280" />
\<Tooltip
contentStyle={{
backgroundColor: "white",
borderRadius: "0.5rem",
border: "1px solid #e5e7eb",
}}
/> <Legend />
\<Bar
dataKey={chart.yAxis}
name={chart.yAxis}
radius={\[4, 4, 0, 0]}
\>
{chart.data.map((\_, index) => (
\<Cell
key={`cell-${index}`}
fill={`hsl(${210 + (index * 30) % 360}, 70%, 50%)`} // ✅ safer palette
/>
))} </Bar> </BarChart> </ResponsiveContainer>
);

```
  case "line":
    return (
      <ResponsiveContainer width="100%" height={300}>
        <LineChart data={chart.data}>
          <CartesianGrid strokeDasharray="3 3" stroke="#e5e7eb" />
          <XAxis dataKey={chart.xAxis} stroke="#6b7280" />
          <YAxis stroke="#6b7280" />
          <Tooltip
            contentStyle={{
              backgroundColor: "white",
              borderRadius: "0.5rem",
              border: "1px solid #e5e7eb",
            }}
          />
          <Legend />
          <Line
            type="monotone"
            dataKey={chart.yAxis}
            stroke="hsl(217, 91%, 60%)"
            strokeWidth={2}
            activeDot={{ r: 8 }}
            name={chart.yAxis}
          />
        </LineChart>
      </ResponsiveContainer>
    );

  case "table":
    return (
      <div className="overflow-x-auto rounded-lg border border-gray-200">
        <table className="min-w-full divide-y divide-gray-200">
          <thead className="bg-gray-50">
            <tr>
              {columns.map((col) => (
                <th
                  key={col}
                  className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
                >
                  {col}
                </th>
              ))}
            </tr>
          </thead>
          <tbody className="bg-white divide-y divide-gray-200">
            {chart.data.slice(0, 10).map((row, rowIndex) => (
              <tr key={rowIndex} className="hover:bg-gray-50">
                {columns.map((col) => (
                  <td
                    key={col}
                    className="px-4 py-3 whitespace-nowrap text-sm text-gray-700"
                  >
                    {String(row[col] ?? "")} {/* ✅ safe render */}
                  </td>
                ))}
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    );

  default:
    return <div>Unsupported visualization type</div>;
}
```

};

return ( <div className="min-h-screen bg-gradient-to-br from-indigo-50 to-purple-50 p-4 md:p-6">
{/\* ... UI code stays same ... \*/} </div>
);
};

export default DashboardCreator;
