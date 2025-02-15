# Como gerar pdf com php
- Baixar a biblioteca TCPDF

```php
    // Estrutura básica
    <?php 
    require_once("TCPDF/tcpdf.php");
    $pdf = new TCPDF();
    $title = "PDF title";
    // PAGE 1
    $pdf->AddPage();
    $html = "
        <table>
            <tr>
                <th>Conteúdo</th>
                <td>{$conteudo}</td>
            </tr>
            <tr>
                <th>Conteúdo 2</th>
                <td>R$ {$conteudo2}</td>
            </tr>
        </table>
    ";
    // ln: procesa tags html
    // fill: desativa a quebra de linha automática 
    // reseth: permite o uso de CSS
    $pdf->writeHTML($html, $ln = true, $fill = false, $reseth = true, $cell = false, $align = '');

    // PAGE 2
    $pdf->AddPage();
    $html = "Página 2";
    $pdf->writeHTML($html, $ln = true, $fill = false, $reseth = true, $cell = false, $align = '');

    $pdf->Output($name = "{$title} fatura.pdf", $dest = 'I');
?>
```