//조립 kit 발행

using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Globalization;
using System.Text;
using System.Linq;
using System.Threading.Tasks;
using System.Windows.Forms;
using DevExpress.Utils;
using DevExpress.XtraEditors;
using DevExpress.XtraGrid.Columns;
using DevExpress.XtraGrid.Views.Base;
using DevExpress.XtraGrid.Views.Grid.ViewInfo;
using DevExpress.XtraLayout;
using HCM.CONTROL;

namespace HCM.TFM
{
    public partial class TFM_PRD_ASSY_KIT_LABEL_PRINT : BaseForm
    {
        #region ================= 생성자 및 전역변수 =================

        HCM.CONTROL.DataService service = null;

        public TFM_PRD_ASSY_KIT_LABEL_PRINT()
        {
            InitializeComponent();

            try
            {
                //컨트롤 이벤트 등록
                CreateControlEvent();

                GridNames = new string[] { grdMain.Name };
            }
            catch (Exception ex)
            {
                CommonHelper.SetLogAndMessage(this.Name, ex.Message);
            }
        }

        /// <summary>
        /// 화면 Load
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        void Form_Load(object sender, EventArgs e)
        {
            try
            {
                //기본 디자인 작업
                SetDefaultDesign();
            }
            catch (Exception ex)
            {
                CommonHelper.SetLogAndMessage(this.Name, ex.Message);

            }
        }

        #endregion ================= 생성자 및 전역변수 =================

        #region ================= 그리드 이벤트 =================

        void Grid_CellValueChanged(object sender, CellValueChangedEventArgs e)
        {
            
        }

        /// <summary>
        /// Master / Detail Grid Last Focused
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        protected void Grid_GotFocus(object sender, EventArgs e)
        {
            FocusedGridControl = sender as DataGridControl;
        }

        private void Grid_CellMerge(object sender, DevExpress.XtraGrid.Views.Grid.CellMergeEventArgs e)
        {
            CONTROL.DataGridView view = sender as CONTROL.DataGridView;
            object objValue1 = view.GetRowCellValue(e.RowHandle1, "ProductionOrderID");
            object objValue2 = view.GetRowCellValue(e.RowHandle2, "ProductionOrderID");

            foreach (string strFieldName in grdMain.MEColumns.Split('|'))
            {
                if (strFieldName.Equals(e.Column.FieldName))
                {
                    if (objValue1.Equals(objValue2))
                    {
                        e.Merge = true;
                        e.Handled = true;
                    }
                    else
                    {
                        e.Merge = false;
                        e.Handled = true;
                    }
                }

                if (e.Column.FieldName.Equals(GridHelper.MultiCheckBoxColumnId))
                {
                    if (objValue1.Equals(objValue2))
                    {
                        e.Merge = true;
                        e.Handled = true;
                    }
                    else
                    {
                        e.Merge = false;
                        e.Handled = true;
                    }
                }
            }
        }

        private void grvMain_RowCellClick(object sender, DevExpress.XtraGrid.Views.Grid.RowCellClickEventArgs e)
        {
            try
            {
                //CONTROL.DataGridView view = sender as CONTROL.DataGridView;

                //if (e.Column.FieldName.Equals(GridHelper.MultiCheckBoxColumnId))
                //{
                //    if (!grdMain.EditYN) return;

                //    DataTable dtView = (view.DataSource as DataView).ToTable();

                //    foreach (DataRow item in dtView.Select("WorkOrderID = '" + view.GetRowCellValue(e.RowHandle, "WorkOrderID") + "'"))
                //    {
                //        if ("Y".Equals(item[GridHelper.MultiCheckBoxColumnId].ToString()))
                //            item[GridHelper.MultiCheckBoxColumnId] = "N";
                //        else
                //            item[GridHelper.MultiCheckBoxColumnId] = "Y";
                //    }

                //    dtView.AcceptChanges();
                //    grdMain.DataSource = dtView;
                //}
            }
            catch (Exception ex)
            {
                CommonHelper.SetLogAndMessage(this.Name, ex.Message);
            }
        }

        void grvMain_RowCellStyle(object sender, DevExpress.XtraGrid.Views.Grid.RowCellStyleEventArgs e)
        {
            string _fieldName = GridHelper.MultiCheckBoxColumnId;

            try
            {
                DevExpress.Skins.Skin currentSkin;
                DevExpress.Skins.SkinElement element;

                string elementName;

                currentSkin = DevExpress.Skins.CommonSkins.GetSkin(this.LookAndFeel);
                
                elementName = DevExpress.Skins.CommonSkins.SkinLayoutItemBackground;
                element = currentSkin[elementName];
                Color skinBackColor = element.Color.BackColor;
                
                DevExpress.XtraGrid.Views.Grid.GridView view = sender as DevExpress.XtraGrid.Views.Grid.GridView;
                
                if (e.Column.FieldName == _fieldName)
                {
                    //e.Appearance.BackColor = ColorInfo.ING_BACK_COLOR_IN_GRID;
                }
                else
                {
                    e.Appearance.BackColor = skinBackColor;
                }
            }
            catch (Exception ex)
            {
                CommonHelper.SetLogAndMessage(this.Name, ex.Message);
            }
        }

        /// <summary>
        /// 멀티 체크
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        /// <param name="_allChecked"></param>
        void grdMain_Event_MultiCheck(object sender, MouseEventArgs e, bool _allChecked)
        {
            string filter = "";
            DataTable dtBindingData = null;
            DataGridControl grid = null;
            try
            {
                grid = sender as DataGridControl;
                if (grid != null)
                {
                    dtBindingData = GetSearchedDataTable(grid.DataSource);

                    if (dtBindingData != null)
                    {
                        filter = "";
                        GridHelper.SetAllCheckBox(_allChecked, dtBindingData, filter);
                    }
                }
            }
            catch (Exception ex)
            {
                CommonHelper.SetLogAndMessage(this.Name, ex.Message);
            }
        }

        #endregion ================= 그리드 이벤트 =================

        #region ================= 컨트롤 이벤트 =================

        /// <summary>
        /// LookupEditControl 데이터에 따른 사이즈 변경
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void LookupEdit_BeforePopup(object sender, EventArgs e)
        {
            LookUpEditControl edit = sender as LookUpEditControl;
            DataView dvEdit = edit.Properties.DataSource as DataView;

            if (dvEdit != null && dvEdit.Count > 0)
            {
                DataTable dtEdit = dvEdit.ToTable();

                edit.Properties.DropDownRows = dtEdit.Rows.Count > 20 ? 20 : dtEdit.Rows.Count;
            }
            else
            {
                DataTable dtEdit = edit.Properties.DataSource as DataTable;

                if (dtEdit != null && dtEdit.Rows.Count > 0)
                    edit.Properties.DropDownRows = dtEdit.Rows.Count > 20 ? 20 : dtEdit.Rows.Count;
            }
        }

        /// <summary>
        /// 엔터키 입력 시 조회 이벤트 실행
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        void txtSearch_KeyDown(object sender, KeyEventArgs e)
        {
            try
            {
                if (e.KeyCode == Keys.Enter)
                {
                    searchButtonControl1_OnSearchAction(null, null);
                }
            }
            catch { }
        }

        /// <summary>
        /// 조회
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void searchButtonControl1_OnSearchAction(object sender, EventArgs e)
        {
            object sEqptLineID = DBNull.Value;

            try
            {
                CommonHelper.DisplayProcessing(true, string.Format(MessageInfo.PROCESSING_GETEXECUTE, new string[] { MessageInfo.SUBWORKORDER }));

                base.OnSearchAction();

                if (cboEquipmentLine.EditValue != null && cboEquipmentLine.EditValue.ToString() != "")
                    sEqptLineID = cboEquipmentLine.EditValue;

                service = new HCM.CONTROL.DataService();
                service.ControllerName = "ProcedureCall";
                service.ActionId = "USP_GetLabelKitPrintTarget";

                service.MultiRecordCreate();
                service.lstMultiParameter.Add("IN_PlanStartDttm", dtpWorkPlan.FromDate);
                service.lstMultiParameter.Add("IN_PlanFinishDttm", dtpWorkPlan.ToDate);
                service.lstMultiParameter.Add("IN_EquipmentLineID", sEqptLineID);               

                service.lstMultiParameter.Add("IN_LanguageID", LoginUserInfo.LanguageID);
                service.AddMultiParameter(service.lstMultiParameter);

                DataTable dtResult = service.StoredProcedureExcute(apiMethodType.GET);

                if (dtResult != null && dtResult.Rows.Count > 0)
                {
                    if (dtResult.Columns.Contains("STATUS"))
                    {
                        settingGridMain();

                        //메시지 출력
                        processMessage = string.Format(MessageInfo.COMPLETE_INQUERY, "0");
                        (base.Parent.TopLevelControl as BaseMDI).SetActionMessage(processMessage);

                        return;
                    }

                    dtResult.Columns.Add(GridHelper.MultiCheckBoxColumnId);
                    dtResult.Columns.Add(GridHelper.RowStateColumnName);

                    dtResult.AcceptChanges();
                    grdMain.DataSource = dtResult;

                    dtResult.TableName = "AssyKitLabelPrinting";

                    // 조회 후 메인그리드의 첫 번째 행을 focus
                    grdMain.Focus();
                    grvMain.FocusedRowHandle = 0;

                    grvMain.BestFitColumns();

                    //메시지 출력
                    processMessage = string.Format(MessageInfo.COMPLETE_INQUERY, new string[] { dtResult.Rows.Count.ToString() });
                    (base.Parent.TopLevelControl as BaseMDI).SetActionMessage(processMessage);
                }
                else
                    (base.Parent.TopLevelControl as BaseMDI).SetActionMessage(string.Format(MessageInfo.NOT_FOUND, MessageInfo.SUBWORKORDER));
            }
            catch (Exception ex)
            {
                CommonHelper.OpenMessageBox(ex.Message, MessageButtonType.OK, MessageIconType.Error);
            }
            finally
            {
                CommonHelper.DisplayProcessing(false);
            }
        }

        /// <summary>
        /// 라벨 발행
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void BtnPrint_Click(object sender, EventArgs e)
        {
            if (CommonHelper.OpenMessageBox(string.Format(MessageInfo.CF000011, btnPrint.Text), MessageButtonType.YesNo, MessageIconType.Question) != System.Windows.Forms.DialogResult.Yes)
                return;

            try
            {
                CommonHelper.DisplayProcessing(true, string.Format(MessageInfo.PROCESSING_SETEXECUTE, btnPrint.Text));

                if (grdMain.DataSource == null || ((DataTable)grdMain.DataSource).Rows.Count == 0)
                {
                    CommonHelper.OpenMessageBox(MessageInfo.NO_CHOSE_DATA, MessageButtonType.OK, MessageIconType.Warning);
                    return;
                }

                DataRow[] drsTarget = ((DataTable)grdMain.DataSource).Select("MULTICHECKBOX = 'Y'");

                DataTable dtTarget = drsTarget == null ? null : drsTarget.Length == 0 ? null : drsTarget.CopyToDataTable();

                if (dtTarget == null || dtTarget.Rows.Count == 0)
                {
                    CommonHelper.OpenMessageBox(MessageInfo.NO_CHOSE_DATA, MessageButtonType.OK, MessageIconType.Warning);
                    return;
                }

                if (dtTarget.Select("IsHold = 'Y'").Length > 0)
                {
                    CommonHelper.OpenMessageBox(string.Format(MessageInfo.EX000020, btnPrint.Text), MessageButtonType.OK, MessageIconType.Warning);
                    return;
                }

                string chkItem = string.Empty;

                foreach (DataRow item in dtTarget.Rows)
                {
                    chkItem += ";" + item["WorkOrderID"];
                }

                chkItem = chkItem.Substring(1);

                DataTable dtPrint = GetLabel(chkItem);

                if (dtPrint == null || dtPrint.Columns.Contains("STATUS") || dtPrint.Rows.Count == 0)
                {
                    CommonHelper.OpenMessageBox(string.Format(MessageInfo.NO_DATA, btnPrint.Text), MessageButtonType.OK, MessageIconType.Warning);
                    return;
                }

                foreach (DataRow row in dtPrint.Rows)
                {
                    CONTROL.CommonPrinter.PrintType = "TCPIP";
                    CONTROL.CommonPrinter.enetPrint(row["LabelScript"].ToString());

                    Common.Delay(1000);
                }

                CommonHelper.OpenMessageBox(MessageInfo.CM000005, MessageButtonType.OK, MessageIconType.Information);
            }
            catch (Exception ex)
            {
                CommonHelper.OpenMessageBox(ex.Message, MessageButtonType.OK, MessageIconType.Error);
            }
            finally
            {
                CommonHelper.DisplayProcessing(false);
            }
        }

        #endregion ================= 컨트롤 이벤트 =================

        #region ================= 사용자 메소드 =================

        /// <summary>
        /// 컨트롤 이벤트 등록
        /// </summary>
        private void CreateControlEvent()
        {
            //Form Load
            this.Load += Form_Load;

            //grid
            this.grvMain.CellValueChanged += Grid_CellValueChanged;
            this.grvMain.CellMerge += Grid_CellMerge;
            this.grvMain.RowCellClick += grvMain_RowCellClick;
            this.grvMain.RowCellStyle += grvMain_RowCellStyle;

            //this.grdMain.Event_MultiCheck += grdMain_Event_MultiCheck;

            //Focus 가지고 있던 Grid 가져오기 위해 사용
            grdMain.GotFocus += Grid_GotFocus;

            //grid 추가/삭제 contextmenu 사용등록
            base.SetContextMenuByGrid(grdMain);

            this.searchButtonControl1.OnSearchAction += searchButtonControl1_OnSearchAction;

            this.btnPrint.Click += BtnPrint_Click;

        }

        /// <summary>
        /// Form 기본 UI 작업
        /// </summary>
        private void SetDefaultDesign()
        {
            try
            {
                settingGridMain();

                //설비라인
                getEquipmentLineInfo();

                dtpWorkPlan.InitCurrDate();

                //dtpWorkPlan.FromDate = Common.ServerTime.AddMonths(-1).ToString("yyyy-MM-dd");
                //dtpWorkPlan.ToDate = Common.ServerTime.ToString("yyyy-MM-dd");

            }
            catch (Exception ex)
            {
                throw new Exception(ex.Message);
            }
        }

        private void settingGridMain()
        {
            DataTable dtResult = new DataTable("AssyKitLabelPrinting");

            foreach (GridColumn col in grvMain.Columns)
            {
                dtResult.Columns.Add(col.FieldName, col.ColumnType);
            }

            if (grdMain.UseMultiCheck && !dtResult.Columns.Contains(GridHelper.MultiCheckBoxColumnId))
                dtResult.Columns.Add(GridHelper.MultiCheckBoxColumnId);

            if (grdMain.UseRowState && !dtResult.Columns.Contains(GridHelper.RowStateColumnName))
                dtResult.Columns.Add(GridHelper.RowStateColumnName);

            dtResult.AcceptChanges();
            grdMain.DataSource = dtResult;
            grdMain.RefreshDataSource();

            if (!grdMain.EditYN)
                grdMain.togglesEditYN();

            base.OnGridStruSearchAction(grdMain, this.Name);
        }

        private DataTable GetLabel(string chkItem)
        {
            string strLabelGubun = string.Empty;
           
            service = new HCM.CONTROL.DataService();
            service.ControllerName = "ProcedureCall";
            service.ActionId = "USP_GetLabelScript";

            service.MultiRecordCreate();
            service.lstMultiParameter.Add("IN_LabelID", "MaterialKITLabel");
            service.lstMultiParameter.Add("IN_LotID", chkItem);
            service.lstMultiParameter.Add("IN_LanguageID", LoginUserInfo.LanguageID);
            service.AddMultiParameter(service.lstMultiParameter);

            return service.StoredProcedureExcute(apiMethodType.GET);
        }

        /// <summary>
        /// 설비 라인 조회
        /// </summary>
        private void getEquipmentLineInfo()
        {

            service = new HCM.CONTROL.DataService();

            //수집항목
            service.ControllerName = "EquipmentLine";
            service.AddParamter("AreaID", "A020", FilterType.Equal);

            DataTable dtCode = service.getDataList();
            dtCode.TableName = service.ControllerName;

            if (dtCode != null && dtCode.Rows.Count > 0)
            {
                dtCode = dtCode.DefaultView.ToTable(true, new string[] { "EquipmentLineID", "EquipmentLineName" });
                CommonHelper.SetDataTableToLookUpEdit(cboEquipmentLine, dtCode, "EquipmentLineName", "EquipmentLineID", true, new string[] { "EquipmentLineID", "EquipmentLineName" }, new string[] { "EquipmentLineID", "EquipmentLineName" });
            }
        }

        #endregion ================= 사용자 메소드 =================
    }
}