# TodayReception

-----------controller

-
using Microsoft.AspNetCore.Mvc;
using Application.CallCenter.TodayReception.Responses;
using Domain.CallCenter.TodayReception.Repository;
using Domain.CallCenter.CompleteClass.Repository;

namespace Application.CallCenter.TodayReception.Controllers
{
    [Route("api/todayReception")]
    [ApiController]
    public class TodayReceptionController : ControllerBase
    {
        private readonly ITodayReception _todayReception;
        private readonly ICompleteClass _completeClass;

        public TodayReceptionController(ITodayReception todayReception, ICompleteClass completeClass)
        {
            _todayReception = todayReception;
            _completeClass = completeClass;
        }

        #region 本日応対情報取得
        /// <summary>
        /// 本日応対情報取得
        /// 使用画面：本日応対
        /// </summary>
        /// <param name="operatorId">ログイン担当者コード</param>
        /// <param name="mode">モード（true：受付分全て表示、false：担当顧客分のみ表示）</param>
        /// <returns></returns>
        /// <remarks>本日応対情報を取得します。</remarks>
        [HttpGet("detail")]
        public ActionResult<IEnumerable<todayReceptionResponse>> findByOperatorIdAndMode([FromQuery] string operatorId, [FromQuery] bool mode)
        {
            try
            {
                var rtn = _todayReception.findByOperatorIdAndMode(operatorId, mode);
                return rtn == null ? null : todayReceptionResponse.ToResponse(rtn).ToList();
            }
            catch (Exception ex)
            {
                return StatusCode(500, ex.Message);
            }
        }
        #endregion

        #region 完了区分コンボボックス情報取得
        /// <summary>
        /// 完了区分コンボボックス情報取得
        /// </summary>
        /// <returns></returns>
        /// <remarks>完了区分（JS）のキーコードと名称１を取得します。</remarks>
        [HttpGet("completeClass")]
        public ActionResult<IEnumerable<completeClassResponse>> findCdAndNm()
        {
            try
            {
                var rtn = _completeClass.findCdAndNmForCompleteClass();
                return rtn == null ? null : completeClassResponse.ToResponse(rtn).ToList();
            }
            catch (Exception ex)
            {
                return StatusCode(500, ex.Message);
            }
        }
        #endregion
    }
}
--------------
responese1
----
using Domain.CallCenter.CompleteClass.Entity;

namespace Application.CallCenter.TodayReception.Responses
{
    public class completeClassResponse
    {
        // キーコード
        public string cd1 { get; set; }

        // 名称
        public string nm1 { get; set; }

        public static IEnumerable<completeClassResponse> ToResponse(IEnumerable<completeClass> sourse)
        {
            return from row in sourse
                select new completeClassResponse()
                {
                    cd1 = row.cd1 ?? string.Empty,
                    nm1 = row.nm1 ?? string.Empty

                };
        }
    }
}
\\\\----4
response 2
------
using Domain.CallCenter.TodayReception.Entity;
using Domain.CallCenter.TodayReception.Service;

namespace Application.CallCenter.TodayReception.Responses
{
    public record todayReceptionResponse
    {
        // 顧客名
        public string customerNm { get; set; }

        // 応対時間
        public string? receptionTm { get; set; }

        // 応対内容
        public string? content { get; set; }

        // 顧客用件
        public string? customerRequire { get; set; }

        // CFS対応
        public string? cfsContent { get; set; }

        // 完了
        public string? complete { get; set; }

        // 担当者
        public string? operatorNm { get; set; }

        // 顧客コード
        public string? customerCd { get; set; }

        // 応対No
        public string? respondNo { get; set; }

        // 完了分類
        public string? completeClass { get; set; }

        // 完了フラグ
        public bool? completeFlg { get; set; }

        // 留守フラグ
        public bool? deliveryFlg { get; set; }

        // 論理削除フラグ
        public bool? datFlg { get; set; }

        public static IEnumerable<todayReceptionResponse> ToResponse(IEnumerable<todayReceptionFindByOperatorIdAndMode> sourse)
        {
            var todayReceptionService = new todayReceptionService();

            return from row in sourse
                   select new todayReceptionResponse()
                   {
                       customerNm = row.custNm.Trim() == ""? row.custNmKn : row.custNm,
                       receptionTm = row.respStartTm ?? string.Empty,
                       content = row.respContentNm ?? string.Empty,
                       customerRequire = row.tokResp ?? string.Empty,
                       cfsContent = row.cfsContent ?? string.Empty,
                       complete = todayReceptionService.completeToModel(row.completeClass, row.completeNm, row.deliveryFlg),
                       operatorNm = row.tanNm ?? string.Empty,
                       customerCd = row.tokCd ?? string.Empty,
                       respondNo = row.respNo ?? string.Empty,
                       completeClass = row.completeClass ?? string.Empty,
                       completeFlg = row.compFlg == null? false : todayReceptionService.completeFlgToModel(row.compFlg),
                       deliveryFlg = row.deliveryFlg == null? false : todayReceptionService.deliveryFlgToModel(row.deliveryFlg),
                       datFlg = row.datFlg == null? false : todayReceptionService.datFlgToModel(row.datFlg)
                   };
        }
    }
}
----4
entity
---------
namespace Domain.CallCenter.TodayReception.Entity;

public record todayReceptionFindByOperatorIdAndMode
{
    public string? respNo { get; init; }
    public string? tokCd { get; init; }
    public string? custNm { get; init; }
    public string? custNmKn { get; init; }
    public string? respStartTm { get; init; }
    public string? contentCd { get; init; }
    public string? tokResp { get; init; }
    public string? cfsContent { get; init; }
    public string? compFlg { get; init; }
    public string? respContentNm { get; init; }
    public string? tanNm { get; init; }
    public string? deliveryFlg { get; init; }
    public string? completeClass { get; init; }
    public string? completeNm { get; init; }
    public string? datFlg { get; init; }
    public string? respColor { get; init; }
}
----respontory
^---------
using Domain.CallCenter.TodayReception.Entity;

namespace Domain.CallCenter.TodayReception.Repository
{
    public interface ITodayReception
    {
        IEnumerable<todayReceptionFindByOperatorIdAndMode> findByOperatorIdAndMode(string oreratorId, bool mode);
    }
}
^\\\\\\\\--------
service 1
----
namespace Domain.CallCenter.TodayReception.Service
{
    public interface ITodayReceptionService
    {
        string completeToModel(string completeClass, string completeNm, string deliveryFlg);
        bool completeFlgToModel(string value);
        bool deliveryFlgToModel(string value);
        bool datFlgToModel(string value);
    }
}
---service 2
---
using Infrastructure.Constants;

namespace Domain.CallCenter.TodayReception.Service
{
    public class todayReceptionService : ITodayReceptionService
    {
        private const string CST_STATUS_NOCOMP = "未";
        private const string CST_STATUS_COMP = "完了";

        private const string CST_CODE_NOCOMP = "00001";
        private const string CST_CODE_COMP = "00002";

        private const string CST_CODE_DELIVERY = "00001";

        #region 完了分類設定
        /// <summary>
        /// 完了分類名称設定
        /// </summary>
        /// <param name="completeClass"></param>
        /// <param name="completeNm"></param>
        /// <param name="deliveryFlg"></param>
        /// <returns>完了分類名称</returns>
        /// <remarks>
        /// 完了分類が空白でない場合は、完了分類名称をセット
        /// 上記以外の場合
        /// 　留守コードが"00001"の場合、「未」をセット
        /// 　上記以外は「完了」をセット
        /// </remarks>
        public string completeToModel(string completeClass, string completeNm, string deliveryFlg)
        {
            if(completeClass != "" && isNumeric(completeClass))
            {
                return completeNm;
            } else if(deliveryFlg == CST_CODE_DELIVERY)
            {
                return CST_STATUS_NOCOMP;
            } else
            {
                return CST_STATUS_COMP;
            }
        }
        #endregion

        #region 完了コードフラグ設定
        /// <summary>
        /// 完了コードフラグ設定
        /// </summary>
        /// <param name="value(完了コード)"></param>
        /// <returns>true(完了) / false(未)</returns>
        /// <remarks>
        /// 完了コードをbool値で返却する
        /// </remarks>
        public bool completeFlgToModel(string value)
        {
            return value == CST_CODE_COMP ? true : false;
        }
        #endregion

        #region 留守コードフラグ設定
        /// <summary>
        /// 留守コードフラグ設定
        /// </summary>
        /// <param name="value(留守コード)"></param>
        /// <returns>true(留守:00001) / false</returns>
        /// <remarks>
        /// 完了コードをbool値で返却する
        /// </remarks>
        public bool deliveryFlgToModel(string value)
        {
            return value == CST_CODE_DELIVERY ? true : false;
        }
        #endregion

        #region 論理削除フラグ設定
        /// <summary>
        /// 論理削除フラグ設定
        /// </summary>
        /// <param name="value(論理削除フラグ)"></param>
        /// <returns>true(削除) / false(使用中)</returns>
        /// <remarks>
        /// 論理削除フラグをbool値で返却する
        /// </remarks>
        public bool datFlgToModel(string value)
        {
            return value == CommonConst.CST_DATKB.DEL ? true : false;
        }
        #endregion

        #region 数値判定
        // 数値であるか判定する
        // 共通でもってもいい
        private bool isNumeric(string value)
        {
            int iVal = 0;

            return int.TryParse(value, System.Globalization.NumberStyles.Any, null, out iVal);
        }
        #endregion
    }
}
\\\\-----------dataFtoday
\\^^-------------

using System.Data;
using System.Text;
using Oracle.ManagedDataAccess.Client;
using Infrastructure.Constants;
using Domain.CallCenter.TodayReception.Repository;
using Domain.CallCenter.TodayReception.Entity;
using Infrastructure.Database.Context;
using Infrastructure.Database.DbConnect.DBConnect;

namespace Infrastructure.CallCenter.TodayReception
{
    public class EFtodayReceptionRepository : ITodayReception
    {
        private readonly respondCContext _context;
        private readonly tokmtaContext   _contextTokmta;
        private readonly tanmtaContext   _contextTanmta;
        private readonly jdnthaContext   _contextJdntha;
        private readonly namemtaContext  _contextNamemta;

        private const string CST_NAMEMTA_KBN_YOKEN = "45";   // 各種名称情報（用件内容）
        private const string CST_NAMEMTA_KBN_KENTO = "JS";   // 各種名称情報（検討状態）

        public EFtodayReceptionRepository(respondCContext context,
                                          tokmtaContext   tokmtaContext,
                                          tanmtaContext   tanmtaContext,
                                          jdnthaContext   jdnthaContext,
                                          namemtaContext  nameContext)
        {
            _context        = context;
            _contextTokmta  = tokmtaContext;
            _contextTanmta  = tanmtaContext;
            _contextJdntha  = jdnthaContext;
            _contextNamemta = nameContext;
        }

        #region 本日応対情報取得
        public IEnumerable<todayReceptionFindByOperatorIdAndMode> findByOperatorIdAndMode(string operatorId, bool mode)
        {
            string querySub;
            StringBuilder queryAll = new StringBuilder();

            IEnumerable<todayReceptionFindByOperatorIdAndMode> todayReceptionDataReader;

            // データベース接続
            DbManager dbManager = new DbManager();

            try
            {
                querySub = $@"
                            SELECT
                              RES.RESPNO AS RESPNO
                             ,RES.TOKCD AS TOKCD
                             ,TRIM((TRIM(TOK.TOKNMA) || ' ' ||TRIM(TOK.TOKNMB))) AS CUSTNM
                             ,TRIM((TRIM(TOK.TOKNK) || ' ' ||TRIM(TOK.TOKNKB))) AS CUSTNMKN
                             ,(SUBSTR(RES.RESPSTM, 1, 2)|| ':' || SUBSTR(RES.RESPSTM, 3, 2)) AS RESPSTARTTM
                             ,RES.CNTNTCD AS CONTENTCD
                             ,RES.YOKEN AS TOKRESP
                             ,RES.CNTNT AS CFSCONTENT
                             ,RES.COMPFLG AS COMPFLG
                             ,NCNT.NAME1 AS RESPCONTENTNM
                             ,TAN.TANNM AS TANNM
                             ,RES.RUSU AS DELIVERYFLG
                             ,RES.SYMPDTL AS COMPLETECLASS
                             ,NCOM.NAME2 AS COMPLETENM
                             ,TOK.DATKB AS DATFLG
                             ,RES.SYMP AS RESPCOLOR
                            FROM EVE_USR1.RESPOND_C RES,
                                 EVE_USR1.TOKMTA TOK,
                                 EVE_USR1.TANMTA TAN,
                                 (
                                   SELECT
                                     CODE1
                                    ,NAME1
                                   FROM EVE_USR1.NAMEMTA
                                   WHERE DATKB = '{CommonConst.CST_DATKB.EXIST}'
                                     AND KBN = '{CST_NAMEMTA_KBN_YOKEN}'
                                 ) NCNT,
                                 (
                                   SELECT
                                     CODE1
                                    ,NAME1
                                    ,NAME2
                                   FROM EVE_USR1.NAMEMTA
                                   WHERE DATKB = '{CommonConst.CST_DATKB.EXIST}'
                                     AND KBN = '{CST_NAMEMTA_KBN_KENTO}'
                                 ) NCOM
                            WHERE RES.RESPSDT = TO_CHAR(SYSDATE,'YYYYMMDD')
                              AND RES.TOKCD = TOK.TOKCD
                              AND RES.TANCD = TAN.TANCD
                              AND NCNT.CODE1(+) = RES.CNTNTCD
                              AND RES.SYMPDTL = NCOM.CODE1(+)
                            ";

                // メインSQL生成
                queryAll.AppendLine("SELECT DISTINCT A.* ");
                queryAll.AppendLine("FROM ( ");

                // modeがfalseの場合は担当顧客分のみ表示、trueの場合は受付分全て表示
                if (mode)
                {
                    queryAll.AppendLine(querySub);
                    queryAll.AppendLine("AND RES.INPTANCD = :operatorId ");
                    queryAll.AppendLine("UNION ");
                    queryAll.AppendLine(querySub);
                    queryAll.AppendLine("AND RES.TANCD = :operatorId ");
                    queryAll.AppendLine(") A");
                }
                else
                {
                    queryAll.AppendLine(querySub);
                    queryAll.AppendLine("AND RES.TANCD = :operatorId ");
                    queryAll.AppendLine(") A");
                }

                // 応対開始時間の降順でソート
                queryAll.AppendLine("ORDER BY A.RESPSTARTTM DESC ");

                using (OracleCommand cmd = new OracleCommand(queryAll.ToString()))
                {
                    cmd.Connection = dbManager.con;
                    cmd.Parameters.Add("operatorId", operatorId);

                    using (OracleDataReader reader = cmd.ExecuteReader())
                    {
                        todayReceptionDataReader = dbManager.DataReaderToEntities<todayReceptionFindByOperatorIdAndMode>(reader);

                        // 戻り値を返す
                        return todayReceptionDataReader.Select(x => new todayReceptionFindByOperatorIdAndMode()
                        {
                            respNo = x.respNo,
                            tokCd = x.tokCd,
                            custNm = x.custNm,
                            custNmKn = x.custNmKn,
                            respStartTm = x.respStartTm,
                            contentCd = x.contentCd,
                            tokResp = x.tokResp,
                            cfsContent = x.cfsContent,
                            compFlg = x.compFlg,
                            respContentNm = x.respContentNm,
                            tanNm = x.tanNm,
                            deliveryFlg = x.deliveryFlg,
                            completeClass = x.completeClass,
                            completeNm = x.completeNm,
                            datFlg = x.datFlg,
                            respColor = x.respColor
                        });
                    }
                }
            }
            catch (OracleException e)
            {
                Console.WriteLine("本日応対取得：todayReceptionList");
                Console.WriteLine("Code: " + e.ErrorCode + "\n" + "Message: " + e.Message);
                Console.WriteLine("例外処理が発生しました。 システム管理者に連絡してください。");

                return null;
            }
            finally
            {
                //データベースを閉じる
                dbManager.Close();
            }
        }
        #endregion
    }
}
---------datamodel
\^^^^----------
namespace Infrastructure.CallCenter.TodayReception.DataModels
{
    /// <summary>
    /// 本日応対
    /// </summary>
    public partial class todayReceptionDataModel
    {
        public string? respNo { get; init; }
        public string? tokCd { get; init; }
        public string? custNm { get; init; }
        public string? custNmKn { get; init; }
        public string? respStartTm { get; init; }
        public string? contentCd { get; init; }
        public string? tokResp { get; init; }
        public string? cfsContent { get; init; }
        public string? compFlg { get; init; }
        public string? respContentNm { get; init; }
        public string? tanNm { get; init; }
        public string? deliveryFlg { get; init; }
        public string? completeClass { get; init; }
        public string? completeNm { get; init; }
        public string? datFlg { get; init; }
        public string? respColor { get; init; }
    }
}
