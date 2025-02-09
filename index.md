---
layout: post
title: Drawing Board
search_exclude: true
description: Drawing Board
author: Zach
---

## Welcome to Scribble With Users

<table>
    <tr>
        <td><a href="{{site.baseurl}}/index">Home</a></td>
        <td><a href="{{site.baseurl}}/competition">Competitive</a></td>
        <td><a href="{{site.baseurl}}/guess">Guess Game</a></td>
        <td><a href="{{site.baseurl}}/leaderboard">LeaderBoard</a></td>
        <td><a href="{{site.baseurl}}/stats">Statistics</a></td>
        <td><a href="{{site.baseurl}}/about">About Us</a></td>
        <td><a href="{{site.baseurl}}/deploy">Deploy Blog</a></td>
    </tr>
</table>

<div id="app"></div>
<div class="form-container" style="margin-top: 20px;">
    <div class="input-group" style="display: flex; gap: 10px;">
        <input type="text" id="userName" placeholder="Your Name" class="form-input" style="height: 3rem; flex: 1;" required>
        <input type="text" id="drawingName" placeholder="Drawing Name" class="form-input" style="height: 3rem; flex: 1;" required>
        <button onclick="saveDrawing()" class="submit-button" style="height: 3rem;">Send</button>
    </div>
    <div id="message"></div>
</div>
<div id="drawings-list" style="margin-top: 20px;"></div>
<table id="drawingsTable" border="1" style="margin-top: 20px; width: 100%;">
    <thead>
        <tr>
            <th>User</th>
            <th>Drawing Name</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody></tbody>
</table>
<style>
    body {
        background: linear-gradient(145deg, #3674B5, #578FCA, #A1E3F9, #D1F8EF);
    }
</style>
<script>
document.addEventListener('DOMContentLoaded', () => {
    const app = document.querySelector('#app');
    if (!app) {
        console.error('Error: #app container not found. Ensure the div with id "app" is in the HTML.');
        return;
    }
    const toolbar = document.createElement('div');
    toolbar.style.cssText = `
        display: flex;
        justify-content: center;
        align-items: center;
        margin-bottom: 10px;
        background: rgba(255, 255, 255, 0.3);
        padding: 10px;
        border-radius: 10px;
        gap: 10px;
        flex-wrap: wrap;
    `;
    const colorPicker = document.createElement('input');
    colorPicker.type = 'color';
    colorPicker.value = '#000000';
    colorPicker.style.cssText = `
        width: 40px;
        height: 40px;
        border: none;
        cursor: pointer;
    `;
    toolbar.appendChild(colorPicker);
    let currentColor = colorPicker.value;
    let isEraser = false;
    colorPicker.addEventListener('input', () => {
        currentColor = colorPicker.value;
        isEraser = false;
        eraserButton.style.background = 'white';
        eraserButton.style.color = 'black';
    });
    const brushSize = document.createElement('input');
    brushSize.type = 'range';
    brushSize.min = '1';
    brushSize.max = '50';
    brushSize.value = '5';
    brushSize.style.cssText = 'margin: 0 10px;';
    toolbar.appendChild(brushSize);
    const eraserButton = document.createElement('button');
    eraserButton.textContent = 'Eraser';
    eraserButton.style.cssText = `
        background: white;
        color: black;
        border: 2px solid #000;
        padding: 10px;
        border-radius: 5px;
        cursor: pointer;
        font-weight: bold;
    `;
    eraserButton.addEventListener('click', () => {
        isEraser = !isEraser;
        if (isEraser) {
            eraserButton.style.background = 'black';
            eraserButton.style.color = 'white';
        } else {
            eraserButton.style.background = 'white';
            eraserButton.style.color = 'black';
        }
    });
    toolbar.appendChild(eraserButton);
    const undoButton = document.createElement('button');
    undoButton.textContent = 'Undo';
    undoButton.style.cssText = `
        background: #FFC107;
        color: white;
        border: none;
        padding: 10px;
        border-radius: 5px;
        cursor: pointer;
        font-weight: bold;
    `;
    undoButton.addEventListener('click', () => {
        if (undoStack.length > 0) {
            undoStack.pop();
            const lastImage = undoStack.length > 0 ? undoStack[undoStack.length - 1] : null;
            const img = new Image();
            img.src = lastImage || '';
            img.onload = () => {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                ctx.drawImage(img, 0, 0);
            };
        }
    });
    toolbar.appendChild(undoButton);
    const resetButton = document.createElement('button');
    resetButton.textContent = 'Reset';
    resetButton.style.cssText = `
        background: #DC3545;
        color: white;
        border: none;
        padding: 10px;
        border-radius: 5px;
        cursor: pointer;
        font-weight: bold;
    `;
    resetButton.addEventListener('click', () => {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        undoStack = [];
    });
    toolbar.appendChild(resetButton);
    const saveButton = document.createElement('button');
    saveButton.textContent = 'Save Drawing';
    saveButton.style.cssText = `
        background: #28A745;
        color: white;
        border: none;
        padding: 10px;
        border-radius: 5px;
        cursor: pointer;
        font-weight: bold;
    `;
    saveButton.addEventListener('click', () => {
        const drawingData = canvas.toDataURL("image/jpeg");
        const link = document.createElement('a');
        link.download = `drawing.jpeg`;
        link.href = drawingData;
        link.click();
    });
    toolbar.appendChild(saveButton);
    const canvas = document.createElement('canvas');
    canvas.width = 800;
    canvas.height = 600;
    canvas.style.cssText = `
        border: 2px solid black;
        background: white;
        cursor: crosshair;
    `;
    const ctx = canvas.getContext('2d');
    ctx.fillStyle = 'white';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    let drawing = false;
    let undoStack = [];
    canvas.addEventListener('mousedown', (e) => {
        drawing = true;
        ctx.beginPath();
        ctx.moveTo(e.offsetX, e.offsetY);
    });
    canvas.addEventListener('mousemove', (e) => {
        if (drawing) {
            ctx.strokeStyle = isEraser ? 'white' : currentColor;
            ctx.lineWidth = brushSize.value;
            ctx.lineCap = 'round';
            ctx.lineTo(e.offsetX, e.offsetY);
            ctx.stroke();
        }
    });
    canvas.addEventListener('mouseup', () => {
        drawing = false;
        ctx.closePath();
        undoStack.push(canvas.toDataURL());
    });
    canvas.addEventListener('mouseleave', () => {
        drawing = false;
    });
    app.appendChild(toolbar);
    app.appendChild(canvas);
    loadDrawings();
});
function saveDrawing() {
    const userName = document.getElementById('userName').value.trim();
    const drawingName = document.getElementById('drawingName').value.trim();
    if (!userName || !drawingName) {
        showMessage('Please fill in all fields correctly', true);
        return;
    }
    const drawingData = canvas.toDataURL("image/jpeg");
    fetch('http://127.0.0.1:8203/api/save-drawing', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            user_name: userName,
            drawing_name: drawingName,
            drawing: drawingData
        })
    })
    .then(response => response.json())
    .then(data => {
        if (data.message) {
            showMessage('Drawing saved successfully');
            loadDrawings();
        } else {
            showMessage(`Error: ${data.error}`, true);
        }
    })
    .catch(error => {
        console.error('Error:', error);
        showMessage('Error saving drawing', true);
    });
}
function loadDrawings() {
    fetch('http://127.0.0.1:8203/api/get-drawings')
        .then(response => response.json())
        .then(data => {
            const drawingsTableBody = document.querySelector('#drawingsTable tbody');
            drawingsTableBody.innerHTML = '';
            data.drawings.forEach(drawing => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${drawing.user_name}</td>
                    <td class="drawing-name">${drawing.drawing_name}</td>
                    <td>
                        <button onclick="editDrawing(this, ${drawing.id})">Edit</button>
                        <button onclick="deleteDrawing(${drawing.id})">Delete</button>
                    </td>
                `;
                drawingsTableBody.appendChild(row);
            });
        })
        .catch(error => {
            console.error('Error:', error);
        });
}
function editDrawing(button, drawingId) {
    const row = button.parentElement.parentElement;
    const drawingNameCell = row.querySelector('.drawing-name');
    const newDrawingName = prompt("Edit drawing name:", drawingNameCell.textContent);
if (!newDrawingName) return;
fetch(`http://127.0.0.1:8203/api/update-drawing/${drawingId}`, {
        method: 'PUT',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({ drawing_name: newDrawingName })
    })
    .then(response => response.json())
    .then(data => {
        if (data.message) {
            drawingNameCell.textContent = newDrawingName;
            showMessage('Drawing name updated successfully');
        } else {
            showMessage(`Error: ${data.error}`, true);
        }
    })
    .catch(error => {
        console.error('Error:', error);
        showMessage('Error updating drawing name', true);
    });
}
function deleteDrawing(drawingId) {
    fetch(`http://127.0.0.1:8203/api/delete-drawing/${drawingId}`, {
        method: 'DELETE',
        headers: {
            'Content-Type': 'application/json'
        }
    })
    .then(response => response.json())
    .then(data => {
        if (data.message) {
            showMessage('Drawing deleted successfully');
            loadDrawings();
        } else {
            showMessage(`Error: ${data.error}`, true);
        }
    })
    .catch(error => {
        console.error('Error:', error);
        showMessage('Error deleting drawing', true);
    });
}
function showMessage(message, isError = false) {
    const messageEl = document.getElementById('message');
    messageEl.style.backgroundColor = isError ? '#fee2e2' : '#dcfce7';
    messageEl.style.color = isError ? '#dc2626' : '#16a34a';
    messageEl.textContent = message;
    setTimeout(() => messageEl.textContent = '', 3000);
}
</script>
