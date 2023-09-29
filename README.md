<?php

namespace App\Http\Controllers;

use Illuminate\Foundation\Auth\Access\AuthorizesRequests;
use Illuminate\Foundation\Validation\ValidatesRequests;
use Illuminate\Routing\Controller as BaseController;
use Illuminate\Support\Facades\DB;

class Controller extends BaseController
{
    use AuthorizesRequests, ValidatesRequests;
    public function tt22pb05(){
        $conditions = DB::select('SELECT ItemID FROM report_pim_22_pb05');
        foreach($conditions as $condition){
            $this->updateTT22PB05($condition->ItemID);
        }
        $master  = DB::select('SELECT * FROM sys_report WHERE ReportID = 2405');
        $detail = DB::select('SELECT * FROM report_pim_22_pb05');
        return response()->json(['master' => $master, 'detail'=> $detail]);
    }
    
    public function updateTT22PB05($condition)
    {
    $sqlUpdate = "UPDATE report_pim_22_pb05 AS MAIN JOIN
    (SELECT ItemID, COUNT(*) AS SUM, SUM(I1) AS I1,SUM(I2) AS I2,SUM(I3) AS I3,SUM(I4) AS I4,SUM(I5) AS I5,SUM(I6) AS I6 FROM
(SELECT
    
    PSI.StatusID AS StatusID,
    PSI.StatusValue AS StatusValue,
		SUM(CASE WHEN A.CateValue = 2 AND PC.CateValue = 1 THEN 1 ELSE 0 END) AS I1,
    SUM(CASE WHEN A.CateValue = 2 AND PC.CateValue = 2 THEN 1 ELSE 0 END) AS I2,
    SUM(CASE WHEN A.CateValue = 2 AND PC.CateValue = 3 THEN 1 ELSE 0 END) AS I3,
    SUM(CASE WHEN A.CateValue = 1 AND PC.CateValue = 1 THEN 1 ELSE 0 END) AS I4,
    SUM(CASE WHEN A.CateValue = 1 AND PC.CateValue = 2 THEN 1 ELSE 0 END) AS I5,
    SUM(CASE WHEN A.CateValue = 1 AND PC.CateValue = 3 THEN 1 ELSE 0 END) AS I6,
    CASE
        WHEN PSI.StatusID = 36 AND PSI.StatusValue = 2 THEN 'A#01'
        WHEN PSI.StatusID = 36 AND PSI.StatusValue = 3 THEN 'A#02'
        WHEN PSI.StatusID = 38 AND PSI.StatusValue = 2 THEN 'A#03'
        WHEN PSI.StatusID = 38 AND PSI.StatusValue = 3 THEN 'A#04'
        WHEN PSI.StatusID = 38 AND PSI.StatusValue IN ('2', '3') THEN 'A#05#01'
        WHEN PSI.StatusID = 38 AND PSI.StatusValue = 4 THEN 'A#05#02'
        WHEN PSI.StatusID = 24 AND PSI.StatusValue = 4 THEN 'A#08'
        WHEN PSI.StatusID = 27 AND PSI.StatusValue IN ('2', '3', '4') THEN 'A#09'
        WHEN PSI.StatusID = 27 AND PSI.StatusValue = 6 THEN 'A#10'
        WHEN PSI.StatusID = 27 AND PSI.StatusValue = 7 THEN 'A#11'
        WHEN PSI.StatusID = 27 AND PSI.StatusValue = 8 THEN 'A#12'
        WHEN PSI.StatusID = 27 AND PSI.StatusValue = 9 THEN 'A#13'
        WHEN PSI.StatusID = 34 AND PSI.StatusValue IN ('3', '4') THEN 'A#14'
        WHEN PSI.StatusID = 35 AND PSI.StatusValue = 1 THEN 'A#15'
    END AS ItemID
FROM
    (
			SELECT
					*
			FROM
					project_cate
			WHERE
					LEFT(CateNo, 4) = '0243'
    ) AS A
    JOIN project_cate AS PC ON A.ProjectID = PC.ProjectID
    AND LEFT(PC.CateNo, 3) = '025'
    JOIN project_status_item AS PSI ON PSI.ProjectID = A.ProjectID AND PSI.StatusID = '34' AND PSI.StatusValue IN ('3', '4')
GROUP BY PSI.StatusID,PSI.StatusValue, PC.CateValue,PC.CateNo, ItemID) AS DB
    GROUP BY DB.ItemID) AS TMP ON MAIN.ItemID = TMP.ItemID
    SET MAIN.I1 = TMP.I1, MAIN.I2 = TMP.I2,  MAIN.I3 = TMP.I3, MAIN.I4 = TMP.I4, MAIN.I5 = TMP.I5, MAIN.I6 = TMP.I6, MAIN.Sumup = TMP.SUM;";
    DB::statement($sqlUpdate);
}

}
