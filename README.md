# Burnup
<form>
    <center>
    <table border="1" style="border-collapse: collapse;width:80%;table-layout:fixed">
        <tr>
            <td>U-235</td>
            <td>U-236</td>
            <td>U-238</td>
            <td>Np-239</td>
            <td>Pu-239</td>
            <td>Pu-240</td>
            <td>Pu-241</td>
            <td>Pu-242</td>
        </tr>
        <tr id="init-val">
            <td><input type="text" id="n1" value="1.0" style="width:70%">kg</td>
            <td><input type="text" id="n2" value="0.0" style="width:70%">kg</td>
            <td><input type="text" id="n3" value="1.0" style="width:70%">kg</td>
            <td><input type="text" id="n4" value="0.0" style="width:70%">kg</td>
            <td><input type="text" id="n5" value="0.0" style="width:70%">kg</td>
            <td><input type="text" id="n6" value="0.0" style="width:70%">kg</td>
            <td><input type="text" id="n7" value="0.0" style="width:70%">kg</td>
            <td><input type="text" id="n8" value="0.0" style="width:70%">kg</td>
        </tr>
        </table>
    中子通量密度<input type="text" id="phi" value="1e18"/>中子/cm<sup>2</sup>s  
    &nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp
    功率<input type="text" id="power" value="5" disabled/>Mw<br/>
    最大时间<input type="text" id="Tmax" value="100"/>天
    &nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp
    时间精度<input type="text" id="delta" value="0.01"/>天<br/>
    模式：恒中子<input type="radio" name="mode" id="mode1" checked/>
    恒功率<input type="radio" name="mode" id="mode2"/>
    &nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp
    方法：解析解<input type="radio" name="method" id="method1" checked/>  
    数值解<input type="radio" name="method" id="method2"/><br/>
    <button>提交</button>
        <button type="reset">重置</button></center>
</form>
<center><div id="fig" style="width:600px;height:250px;"></div></center>
<center><table id="table"></table></center>
<script src="https://cdn.plot.ly/plotly-1.2.0.min.js"></script>
<script>
    (function(){
        const submitButton = document.querySelector("form > center > button");
        const initValTags = Array.prototype.slice.call(document.querySelectorAll("#init-val td input"));
        const n1tag = document.querySelector("#mode1");
        const n2tag = document.querySelector("#mode2");
        const m1tag = document.querySelector("#method1");
        const m2tag = document.querySelector("#method2");
        const phitag = document.querySelector("#phi");
        const powerTag = document.querySelector("#power");
        n1tag.addEventListener("click", (event) => {
            powerTag.setAttribute("disabled", "true");
            phitag.disabled=false;
        })
        n2tag.addEventListener("click", (event) => {
            phitag.setAttribute("disabled", "true");
            powerTag.disabled=false;
        })
        submitButton.addEventListener("click", (event) => {
            event.preventDefault();
            var Figure = document.querySelector("#fig");
            var table = document.querySelector("#table");
            table.innerHTML = "";
            const initVals = initValTags.map(tag => tag.value ? Number(tag.value) : 0.0);
            var masscoef = [2.56e24,2.55e24,2.53e24,2.52e24,2.52e24,2.51e24,2.50e24,2.49e24];
            const tMax = Number(document.querySelector("#Tmax").value);
            let neutronConservation = n1tag.checked;
            let analyticalSolution = m1tag.checked;
            let phi = Number(document.querySelector("#phi").value)*8.64e-20;
            let delta = Number(document.querySelector("#delta").value);
            let power = Number(document.querySelector("#power").value);
            max = Math.floor(tMax/delta);
            var n = new Array();
            var T = new Array();
            for (var i = 0; i < max; i++) {
                T.push(i*delta);
            }
            for (let i = 0; i < 8; i++) {
                n[i] = new Array();
                n[i].push(initVals[i]*masscoef[i]);
            }
            if (neutronConservation) {
                if (analyticalSolution) {
                    for (var i = 0; i < max; i++) {
                        n[0][i] = n[0][0] * Math.exp(-629.0*phi*i*delta);
                        n[1][i] = n[1][0] + 0.161 * n[0][0] * (1 - Math.exp(-629.0*phi*i*delta));
                        n[2][i] = n[2][0] * Math.exp(-2.73*phi*i*delta);
                        n[3][i] = n[3][0] * Math.exp(-0.0124*i*delta) + n[2][0] * (Math.exp(-0.0124*i*delta) - Math.exp(-2.73*phi*i*delta)) / (phi - 4.54e-3) * phi;
                        n[4][i] = n[4][0] * Math.exp(-1016.5*phi*i*delta) + n[3][0] * (0.0000121987 * Math.exp(-0.0124*i*delta) - 0.0000121987 * Math.exp(-1016.5*phi*i*delta))/(-0.0000121987 + phi) + n[2][0] * (Math.exp(-2.73*phi*i*delta) * (1.4921e-10 - 0.0000122316 *phi) + Math.exp(-1016.5*phi*i*delta) * (-1.4921e-10 + 3.28502e-8*phi) + 0.0000121987 * Math.exp(-0.0124*i*delta)*phi)/(5.54081e-8 + phi * (-0.00455432 +phi));
                        n[5][i] = n[5][0] * Math.exp(-286*phi*i*delta) + n[4][0] * (-0.375 * Math.exp(-1016.5*phi*i*delta) + 0.375 * Math.exp(-286*phi*i*delta)) + n[3][0] * (Math.exp(-286*phi*i*delta) * (1.98e-10 - 0.00001626 * phi) + Math.exp(-1016.5*phi*i*delta) * (-1.98e-10 + 4.576e-6 * phi) +  0.00001169 * Math.exp(-0.0124*i*delta) * phi)/(5.289e-10 + phi * (-0.00005556 + phi)) + n[2][0] * (0.0000116869 * Math.exp(-0.0124*i*delta) * phi*phi + Math.exp(-2.73*phi*i*delta) * (-6.26e-15 + (6.57e-10 -  0.0000118313 *phi) *phi) + Math.exp(-1016.5*phi*i*delta) * (-2.42651e-15 + (5.65006e-11 - 1.23216e-8 *phi) *phi) + Math.exp(-286*phi*i*delta) * (8.68403e-15 + (-7.13792e-10 + 1.56729e-7 *phi) *phi))/(-2.40231e-12 + phi * (2.52868e-7 + phi * (-0.00459768 + phi)));
                        n[6][i] = n[6][0] * Math.exp(-1434*phi*i*delta) + n[5][0] * 0.249 * (Math.exp(-286*phi*i*delta)-Math.exp(-1434*phi*i*delta)) + n[4][0] * (0.1635 * Math.exp(-1434*phi*i*delta) - 0.256945 * Math.exp(-1016.5*phi*i*delta) + 0.0934447 * Math.exp(-286*phi*delta*i)) + n[3][0] * (-((4.05145e-6 * Math.exp(-286*phi*i*delta))/(-0.0000433566 + phi)) + (3.1344e-6 *Math.exp(-1016.5*phi*i*delta))/(-0.0000121987 + phi) - (1.41381e-6 *Math.exp(-1434*phi*i*delta))/(-8.64714e-6 + phi) + (Math.exp(-0.0124*i*delta) * (2.95823e-31 + (-1.29247e-26 + 2.33086e-6 *phi) *phi))/(-4.57343e-15 + phi * (1.00929e-9 + phi * (-0.0000642025 + phi)))) + n[2][0] * (Math.exp(-0.0124*i*delta) * phi * (2.95823e-31 + (-1.29247e-26 + 2.33086e-6 *phi) *phi) + Math.exp(-2.73*phi*i*delta) * (1.08123e-20 + phi * (-2.38612e-15 + (1.51785e-10 - 2.36416e-6 *phi) *phi)) + Math.exp(-1016.5*phi*i*delta) * (1.43736e-20 + phi * (-1.99692e-15 + (3.87776e-11 - 8.44068e-9 *phi) *phi)) + Math.exp(-1434*phi*i*delta)*(-6.4783e-21 + phi* (6.81909e-16 + (-1.23985e-11 + 2.6967e-9 * phi) *phi)) + Math.exp(-286*phi*i*delta) * (-1.87076e-20 + phi * (3.70113e-15 + (-1.78164e-10 + 3.90456e-8 *phi) *phi)))/(2.07731e-17 + phi * (-4.5889e-12 + phi * (2.92625e-7 + phi * (-0.00460633 + phi))));
                        n[7][i] = n[7][0] + n[6][0] * 0.296 * (1-Math.exp(-1434*phi*i*delta)) + n[5][0] * (0.296374 + 0.0738353 * Math.exp(-1434*phi*i*delta) - 0.370209 * Math.exp(-286*phi*i*delta)) + n[4][0] * (0.0798883 - 0.0484572 * Math.exp(-1434*phi*i*delta) + 0.107429 * Math.exp(-1016.5*phi*i*delta) - 0.13886 * Math.exp(-286*phi*i*delta)) + n[3][0] * (0.0798883 + (6.02051e-6 *Math.exp(-286*phi*i*delta))/(-0.0000433566 + phi) - (1.3105e-6 *Math.exp(-1016.5*phi*i*delta))/(-0.0000121987 + phi) + (4.19016e-7 *Math.exp(-1434.*phi*i*delta))/(-8.64714e-6 + phi) + (Math.exp(-0.0124*i*delta) * (-7.39557e-32 + phi* (9.69352e-27 + (-7.94093e-22 - 0.0798883 *phi) *phi)))/(-4.57343e-15 + phi * (1.00929e-9 + phi* (-0.0000642025 + phi)))) + n[2][0] * (1.65953e-18 + Math.exp(-0.0124*i*delta) * phi * (-7.39557e-32 + phi * (9.69352e-27 + (-7.94093e-22 - 0.0798883 * phi) * phi)) + Math.exp(-286*phi*i*delta) * (2.77998e-20 + phi * (-5.49994e-15 + (2.64754e-10 - 5.80223e-8 * phi) * phi)) + Math.exp(-1434*phi*i*delta) * (1.92e-21 + phi * (-2.021e-16 + (3.6746e-12 - 7.9923e-10 * phi) * phi)) + Math.exp(-1016.5*phi*i*delta) * (-6.00961e-21 + phi * (8.34914e-16 + (-1.6213e-11 + 3.52906e-9 * phi) * phi)) + Math.exp(-2.73*phi*i*delta) * (-1.68324e-18 + phi * (3.71466e-13 + (-2.36295e-8 +  0.000368047 * phi) * phi)) + phi * (-3.66599e-13 + phi * (2.33773e-8 + (-0.000367991 + 0.0798883 * phi) * phi)))/(2.07731e-17 + phi * (-4.5889e-12 + phi * (2.92625e-7 + phi * (-0.00460633 + phi))));
                        for (var j = 0; j < 8; j++) {
                            if (n[j][i]<0) n[j][i] = 0;
                        }
                    }
                }
                else {
                    for (var i = 1; i < max; i++) {
                        n[0].push(n[0][i-1]*(1.0 - 629.0* phi * delta));
                        n[1].push(n[1][i-1] + n[0][i-1] * 101.1 * phi *  delta);
                        n[2].push(n[2][i-1]*(1.0 - 2.73 * phi * delta));
                        n[3].push(n[3][i-1]*(1.0 - 1.24e-2 * delta) + n[2][i-1] * 2.73 * phi * delta);
                        n[4].push(n[4][i-1]*(1.0 - 1016.5 * phi * delta) + n[3][i-1] * 1.24e-2 * delta);
                        n[5].push(n[5][i-1]*(1.0 - 286.0 * phi * delta) + n[4][i-1] * 274.0 * phi * delta);
                        n[6].push(n[6][i-1]*(1.0 - 1434.0 * phi * delta) + n[5][i-1] * 286.0 * phi * delta);
                        n[7].push(n[7][i-1] + n[6][i-1] * 425.0 * phi * delta); 
                    }
                }
            }
            else {
                if (analyticalSolution) {
                    alert("该限制条件尚无解析解");
                    return;
                }
                else {
                    for (var i = 1; i < max; i++) {
                        phi = power / (528*200*n[0][i-1]+14*210*n[4][i-1]+1009*210*n[6][i-1]) * 5.39e23;
                        n[0].push(n[0][i-1]*(1.0 - 629.0* phi * delta));
                        n[1].push(n[1][i-1] + n[0][i-1] * 101.1 * phi *  delta);
                        n[2].push(n[2][i-1]*(1.0 - 2.73 * phi * delta));
                        n[3].push(n[3][i-1]*(1.0 - 1.24e-2 * delta) + n[2][i-1] * 2.73 * phi * delta);
                        n[4].push(n[4][i-1]*(1.0 - 1016.5 * phi * delta) + n[3][i-1] * 1.24e-2 * delta);
                        n[5].push(n[5][i-1]*(1.0 - 286.0 * phi * delta) + n[4][i-1] * 274.0 * phi * delta);
                        n[6].push(n[6][i-1]*(1.0 - 1434.0 * phi * delta) + n[5][i-1] * 286.0 * phi * delta);
                        n[7].push(n[7][i-1] + n[6][i-1] * 425.0 * phi * delta); 
                        for (var j = 0; j < 8; j++) {
                            if (n[j][i]<0) n[j][i] = 0;
                        }
                    }
                }
            }
			for(var i = 0; i < max; i++) {
                for(var j = 0; j < 8; j++) {
                    n[j][i] = (n[j][i]/masscoef[j]);
                }
            }
            Plotly.newPlot(Figure, [
                {x: T,  y: n[0], name: 'U-235'}, 
                {x: T,  y: n[1], name: 'U-236'}, 
                {x: T,  y: n[2], name: 'U-238'}, 
                {x: T,  y: n[3], name: 'Np-239'}, 
                {x: T,  y: n[4], name: 'Pu-239'}, 
                {x: T,  y: n[5], name: 'Pu-240'}, 
                {x: T,  y: n[6], name: 'Pu-241'}, 
                {x: T,  y: n[7], name: 'Pu-242'}, 
                ], {margin: {t: 1}});
            table.setAttribute("border", "1");
            table.setAttribute("style", "border-collapse: collapse;")
            var isotopes = ["U-235", "U-236", "U-238", "Np-239", "Pu-239", "Pu-240", "Pu-241", "Pu-242"];
            var tr1 = document.createElement("tr");
            var td1 = document.createElement("td");
            td1.innerHTML = "时间";
            tr1.appendChild(td1)
            for(var i = 0; i < 8; i++) {
                var td = document.createElement("td");
                td.innerHTML = isotopes[i];
                tr1.appendChild(td);
            }
            table.appendChild(tr1);
            for(var i = 0; i < max; i++) {
                var tr = document.createElement("tr");
                var td2 = document.createElement("td");
                td2.innerHTML = T[i].toExponential(3);
                tr.appendChild(td2);
                for(var j = 0; j < 8; j++) {
                    var td = document.createElement("td");
                    td.innerHTML = n[j][i].toExponential(4);
                    tr.appendChild(td);
                }
                table.appendChild(tr);
            }
        });
    })();
</script>
