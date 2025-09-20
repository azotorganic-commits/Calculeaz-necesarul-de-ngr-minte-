# Calculeaz-necesarul-de-inngrasaminte
<!DOCTYPE html>
<html lang="ro">
<head>
  <meta charset="UTF-8">
  <title>Calculator Îngrășământ & pH</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-annotation@1.4.0"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f4f6f9;
      padding: 30px;
      color: #333;
      max-width: 900px;
      margin: auto;
    }
    h1 {
      color: #2c6e49;
      text-align: center;
    }
    label {
      display: block;
      margin-top: 15px;
      font-weight: bold;
    }
    input {
      padding: 8px;
      margin-top: 5px;
      width: 200px;
      border: 1px solid #ccc;
      border-radius: 5px;
    }
    button {
      margin-top: 20px;
      padding: 10px 20px;
      background: #2c6e49;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    button:hover {
      background: #1e4c32;
    }
    .rezultat {
      margin-top: 25px;
      padding: 15px;
      background: #fff;
      border-radius: 8px;
      box-shadow: 0 0 5px rgba(0,0,0,0.1);
    }
    .recomandare {
      margin-top: 15px;
      padding: 12px;
      background: #eafbea;
      border-left: 4px solid #2c6e49;
    }
    canvas {
      margin-top: 30px;
    }
  </style>
</head>
<body>
  <h1>Calculator disponibilitate îngrășământ</h1>

  <label for="doza">Doza aplicată (kg/ha):</label>
  <input type="number" id="doza" placeholder="Ex: 100">

  <label for="ph">pH-ul solului:</label>
  <input type="number" step="0.1" id="ph" placeholder="Ex: 6.5">

  <button onclick="calculeaza()">Calculează</button>

  <div class="rezultat" id="rezultat"></div>

  <canvas id="grafic"></canvas>

  <script>
    let chart;

    function calculeaza() {
      let doza = parseFloat(document.getElementById("doza").value);
      let ph = parseFloat(document.getElementById("ph").value);
      let procent_blocat = 0;

      if (isNaN(doza) || isNaN(ph) || doza <= 0) {
        document.getElementById("rezultat").innerHTML = "<p style='color:red;'>Introduceți valori valide!</p>";
        return;
      }

      // Calcul blocaj îngrășământ
      if (ph < 7) {
        procent_blocat = 0.20;
      } else if (ph > 7) {
        procent_blocat = 0.15;
      } else {
        procent_blocat = 0.0;
      }

      let blocat = doza * procent_blocat;
      let disponibil = doza - blocat;

      // Recomandări Terracalco 95 + estimare producție
      let recomandare = "";
      if (ph < 6.5) {
        let diferenta = 7 - ph;
        let doza_terracalco = (diferenta / 0.3).toFixed(1); // estimare simplă
        recomandare = `
          <div class="recomandare">
            <h3>Recomandare corectare pH:</h3>
            <p>Solul este <b>acid</b>. Se recomandă aplicarea de <b>Terracalco 95</b>.</p>
            <p>Estimativ: ~<b>${doza_terracalco} tone/ha</b> pentru a ajunge la pH neutru (7.0).</p>
            <p><b>Beneficiu așteptat:</b> Creștere de producție cu <b>+15–25%</b> după corectarea pH-ului.</p>
          </div>
        `;
      } else if (ph > 8.5) {
        recomandare = `
          <div class="recomandare">
            <h3>Recomandare pentru sol sărăturat:</h3>
            <p>Solul este <b>alcalin și posibil sărăturat</b>. Excesul de sodiu (Na⁺) poate afecta plantele.</p>
            <p>Se recomandă <b>Terracalco 95</b> pentru înlocuirea sodiului din sol și îmbunătățirea structurii.</p>
            <p><b>Beneficiu așteptat:</b> Creștere de producție cu <b>+10–20%</b> după corectarea solului.</p>
          </div>
        `;
      } else {
        recomandare = `
          <div class="recomandare">
            <p>pH-ul solului este aproape optim. Se așteaptă beneficii mici (<b>+0–5%</b>).</p>
          </div>
        `;
      }

      // Afișare rezultate
      document.getElementById("rezultat").innerHTML = `
        <h2>Rezultate:</h2>
        <p>Doza aplicată: <b>${doza.toFixed(2)} kg/ha</b></p>
        <p>pH sol: <b>${ph}</b></p>
        <p>Îngrășământ disponibil: <b>${disponibil.toFixed(2)} kg/ha</b></p>
        <p>Îngrășământ blocat: <b>${blocat.toFixed(2)} kg/ha</b></p>
        ${recomandare}
      `;

      // Construire date pentru grafic (pH 4-9)
      let ph_values = [];
      let blocat_values = [];
      let disponibil_values = [];
      for (let i = 4; i <= 9; i += 0.5) {
        let procent = 0;
        if (i < 7) procent = 0.20;
        else if (i > 7) procent = 0.15;
        else procent = 0;
        ph_values.push(i.toFixed(1));
        let bloc = doza * procent;
        blocat_values.push(bloc.toFixed(2));
        disponibil_values.push((doza - bloc).toFixed(2));
      }

      // Redesenare grafic
      if (chart) chart.destroy();
      let ctx = document.getElementById("grafic").getContext("2d");
      chart = new Chart(ctx, {
        type: "line",
        data: {
          labels: ph_values,
          datasets: [
            {
              label: "Îngrășământ blocat (kg/ha)",
              data: blocat_values,
              borderColor: "#2c6e49",
              backgroundColor: "rgba(44, 110, 73, 0.2)",
              fill: true,
              tension: 0.3,
              pointRadius: 4
            },
            {
              label: "Îngrășământ disponibil (kg/ha)",
              data: disponibil_values,
              borderColor: "#0077cc",
              backgroundColor: "rgba(0, 119, 204, 0.2)",
              fill: true,
              tension: 0.3,
              pointRadius: 4
            }
          ]
        },
        options: {
          responsive: true,
          plugins: {
            legend: { position: "top" },
            title: { display: true, text: "Efectul pH-ului asupra disponibilității îngrășămintelor" },
            annotation: {
              annotations: {
                liniePH: {
                  type: "line",
                  xMin: ph,
                  xMax: ph,
                  borderColor: "red",
                  borderWidth: 2,
                  label: {
                    enabled: true,
                    content: `pH introdus: ${ph}`,
                    position: "end"
                  }
                }
              }
            }
          },
          scales: {
            x: { title: { display: true, text: "pH sol" } },
            y: { title: { display: true, text: "Cantitate (kg/ha)" } }
          }
        }
      });
    }
  </script>
</body>
</html>
