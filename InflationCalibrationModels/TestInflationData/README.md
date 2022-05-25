# [ReflexInflationCalibratorModels](https://github.com/riskevolution/ReflexDotNetDocs/tree/main/InflationCalibrationModels/README.md) 2022.3.25</Version>

# InflationCalibrationModels.sln

## 12. [Test Inflation Data](https://github.com/riskevolution/ReflexDotNetDocs/tree/main/InflationCalibrationModels/TestInflationData/README.md)
#### [TestInflationData.csproj](https://github.com/riskevolution/ReflexDotNetDocs/tree/main/InflationCalibrationModels/TestInflationData/README.md)

```c#

        public void Calc( DateTime todayDate, IEnumerable<InflationIndex> inflationIndexIds = null, string calculationMode = null )
        {
            eTodayDate = todayDate;

            bool checkInputsOk = Common_CheckInputs.CheckAll(PendingCache, null);

            var otrBase = GetOTRBase(iIndices);
            var otrBaseYm = Common_YearMonth.Convert(otrBase);

            if (!checkInputsOk) throw new System.Exception("Inputs not acceptable");

            var lastHistoric = Common_HistoricPublications.GetLastHistoric(IRCC_Initial);

            Dictionary<InflationIndexIdx, YearMonth> lastProjected = Common_HistoricPublications.GetLastProjected(PendingCache, IRCC_Initial.InflationIndexValues);

            Dictionary<InflationIndexIdx, Dictionary< YearMonth, double>> inflationIndexValues = Common_ExtendCurve.Extend(IRCC_Initial.InflationIndexValues, lastProjected);

            var outrights = PendingCache.PendingOutrights();

            outrights = Common_InterpOnTheRun.Interp( outrights, otrBaseYm );

            var upliftDeltas = Common_CalcDeltaUplifts.GetDeltaUpliftNodes(inflationIndexValues, lastHistoric, outrights);

            var upliftsDeltasInterp = Common_CalcDeltaUplifts.Interp(upliftDeltas, lastHistoric);

            var fixingQUotes = Common_InterpOnTheRun.AllButOnTheRun(PendingCache.PendingOutrights(), otrBaseYm);

            Dictionary<InflationIndexIdx, Dictionary<YearMonth, double>> fixingIndexValues = Common_Fixings.CalcIndexValues(fixingQUotes, IRCC_Initial.InflationIndexValues);


            IRCC_Calibrated = Common_UpdateCalibrated.Update(DateTime.UtcNow, otrBaseYm, IRCC_Initial, upliftsDeltasInterp, fixingIndexValues );

            var curveData = new CurveDataIndexValues( IRCC_Calibrated.InflationIndexValues, new Dictionary<InflationIndexIdx, ReflexManaged.YearMonth>());

            CalibT1 = new InflationCalibrationVersionTwo( curveData, CALIB_VERSION_ONE );

            eModelState = RiskEvolution.Inflation.API.EModelState.Calibrated;

            UncalculatedPendingInputs = false;

        }
```
