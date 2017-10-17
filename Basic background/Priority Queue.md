### Piriority Queue

Costs of finding the largest M in a stream of N items
<table>
    <tr>
        <th rowspan="2">Client</th>
        <th colspan="2">order of growth</th>
    </tr>
    <tr>
        <th>time</th>
        <th>space</th>
    </tr>
    <tr>
        <th>sort client</th>
        <th><img src="http://latex.codecogs.com/gif.latex?NlogN"></th>
        <th>N</th>
    </tr>
    <tr>
        <th>PQ client using elementary implementation</th>
        <th>NM</th>
        <th>M</th>
    </tr>
    <tr>
        <th>PQ client using heap-based implementation</th>
        <th><img src="http://latex.codecogs.com/gif.latex?NlogM"></th>
        <th>M</th>
    </tr>
</table>