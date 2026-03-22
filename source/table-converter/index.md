---
title: Markdown-LaTeX Table Converter
date: 2026-03-22 14:49:15
---

<div class="converter-container">
    <div class="converter-row">
        <div class="converter-column">
            <label for="md-input">Markdown Table</label>
            <textarea id="md-input" spellcheck="false" placeholder="| Header 1 | Header 2 |
| --- | --- |
| Cell 1 | Cell 2 |"></textarea>
        </div>
        <div class="converter-column">
            <label for="latex-input">LaTeX Table</label>
            <textarea id="latex-input" spellcheck="false" placeholder="\begin{tabular}{|l|l|}
\hline
Header 1 & Header 2 \\
\hline
Cell 1 & Cell 2 \\
\hline
\end{tabular}"></textarea>
        </div>
    </div>
    <div id="error-message" class="error-text"></div>
</div>

<style>
    .converter-container {
        width: 100%;
        margin-top: 20px;
        font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    }
    .converter-row {
        display: flex;
        gap: 20px;
        margin-bottom: 10px;
    }
    .converter-column {
        flex: 1;
        display: flex;
        flex-direction: column;
    }
    .converter-column label {
        margin-bottom: 8px;
        font-weight: 600;
        font-size: 1.1rem;
    }
    .converter-column textarea {
        height: 400px;
        padding: 12px;
        border: 1px solid #ddd;
        border-radius: 8px;
        font-family: "SFMono-Regular", Consolas, "Liberation Mono", Menlo, monospace;
        font-size: 14px;
        line-height: 1.5;
        resize: vertical;
        background-color: #f9f9f9;
        transition: border-color 0.2s;
    }
    .converter-column textarea:focus {
        outline: none;
        border-color: #A31F34;
        background-color: #fff;
    }
    .error-text {
        color: #e53935;
        margin-top: 15px;
        padding: 10px;
        border-radius: 4px;
        background-color: #ffebee;
        border: 1px solid #ffcdd2;
        font-size: 14px;
        display: none;
    }
    @media (max-width: 768px) {
        .converter-row {
            flex-direction: column;
        }
        .converter-column textarea {
            height: 250px;
        }
    }
</style>

<script>
(function() {
    const mdInput = document.getElementById('md-input');
    const latexInput = document.getElementById('latex-input');
    const errorMsg = document.getElementById('error-message');

    let isUpdating = false;

    mdInput.addEventListener('input', () => {
        if (isUpdating) return;
        isUpdating = true;
        try {
            const val = mdInput.value.trim();
            if (!val) {
                latexInput.value = "";
                hideError();
            } else {
                const latex = mdToLatex(val);
                latexInput.value = latex;
                hideError();
            }
        } catch (e) {
            showError("Markdown Error: " + e.message);
        }
        isUpdating = false;
    });

    latexInput.addEventListener('input', () => {
        if (isUpdating) return;
        isUpdating = true;
        try {
            const val = latexInput.value.trim();
            if (!val) {
                mdInput.value = "";
                hideError();
            } else {
                const md = latexToMd(val);
                mdInput.value = md;
                hideError();
            }
        } catch (e) {
            showError("LaTeX Error: " + e.message);
        }
        isUpdating = false;
    });

    function showError(msg) {
        errorMsg.textContent = msg;
        errorMsg.style.display = 'block';
    }

    function hideError() {
        errorMsg.style.display = 'none';
    }

    function mdToLatex(md) {
        const lines = md.split('\n').map(l => l.trim()).filter(l => l);
        if (lines.length < 2) throw new Error("Table must have at least a header and a separator line.");

        // Find the separator line (e.g., |---|---|)
        const sepIndex = lines.findIndex(l => /^\|?\s*[:\-]+\s*(\|?\s*[:\-]+\s*)*\|?\s*$/.test(l));
        if (sepIndex === -1) throw new Error("Could not find table separator line (e.g., |---|).");

        const parseRow = (row) => {
            let cells = row.split('|').map(c => c.trim());
            if (cells[0] === "") cells.shift();
            if (cells[cells.length - 1] === "") cells.pop();
            return cells;
        };

        const headerRow = parseRow(lines[0]);
        const colCount = headerRow.length;

        // Parse alignment from separator line
        const sepRow = parseRow(lines[sepIndex]);
        const alignments = sepRow.map(cell => {
            const left = cell.startsWith(':');
            const right = cell.endsWith(':');
            if (left && right) return 'c';
            if (right) return 'r';
            return 'l'; // default to left
        });

        // Ensure alignments match colCount
        while (alignments.length < colCount) alignments.push('l');

        let latex = "\\begin{tabular}{" + alignments.map(a => "|" + a).join('') + "|}\n\\hline\n";
        latex += headerRow.join(' & ') + " \\\\\n\\hline\n";

        for (let i = 0; i < lines.length; i++) {
            if (i === 0 || i === sepIndex) continue;
            const row = parseRow(lines[i]);
            if (row.length > 0) {
                // Pad or truncate row to match colCount
                const paddedRow = Array(colCount).fill("");
                for(let j=0; j<colCount && j<row.length; j++) paddedRow[j] = row[j];
                latex += paddedRow.join(' & ') + " \\\\\n\\hline\n";
            }
        }

        latex += "\\end{tabular}";

        return `\\begin{table}[htbp]
    \\centering
    \\caption{title}
    \\label{tab:data}
    ${latex}
\\end{table}`;
    }

    function latexToMd(latex) {
        // Try to find tabular inside table environment, or just tabular
        const tabularMatch = latex.match(/\\begin\{tabular\}\s*\{([^}]*)\}([\s\S]*?)\\end\{tabular\}/);
        if (!tabularMatch) throw new Error("Could not find \\begin{tabular}...\\end{tabular} block.");

        const alignStr = tabularMatch[1].replace(/\|/g, '').trim();
        const alignments = alignStr.split('').map(char => {
            if (char === 'c') return ':---:';
            if (char === 'r') return '---:';
            return ':---';
        });

        const content = tabularMatch[2].trim();
        const rows = content.split('\\\\')
            .map(r => r.replace(/\\hline/g, '').trim())
            .filter(r => r)
            .map(r => r.split('&').map(c => c.trim()));

        if (rows.length === 0) throw new Error("No rows found in LaTeX table.");

        const colCount = rows[0].length;
        
        // Ensure alignments match colCount
        while (alignments.length < colCount) alignments.push(':---');
        const finalAlignments = alignments.slice(0, colCount);

        let md = "| " + rows[0].join(" | ") + " |\n";
        md += "| " + finalAlignments.join(" | ") + " |\n";

        for (let i = 1; i < rows.length; i++) {
            const row = rows[i];
            const paddedRow = Array(colCount).fill("");
            for(let j=0; j<colCount && j<row.length; j++) paddedRow[j] = row[j];
            md += "| " + paddedRow.join(" | ") + " |\n";
        }

        return md;
    }
})();
</script>
