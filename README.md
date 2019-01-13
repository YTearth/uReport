# 清算流水/汇总报表生成
&nbsp;

&nbsp; 
> 通过配置文件与代码生成工具，统一报表格式定义与打印，同时解决常见汇总与归类需求。
报表暂时分为两类：一是统计类，需要根据输入内容进行分类汇总后输出；二是流水类，直接将记录格式化输出。

&nbsp;

&nbsp;


## 1.&nbsp;&nbsp;变量
### 1.1&nbsp;&nbsp;变量定义
#### 1.1.1&nbsp;&nbsp;<instance>
1.1.1 报表实例化对象：<instance> &nbsp;&nbsp;&nbsp;&nbsp;用于初始化报表，通常包含报表名与头部部分的信息；
#### 1.1.2&nbsp;&nbsp;<input-def>
1.1.2 报表输入对象：<input-def> &nbsp;&nbsp;&nbsp;&nbsp;用于接收应用输入的交易明细或汇总信息，可直接用于报表输出或统计计算，对应于目前清算应用的查询结果或中间记录；
#### 1.1.3&nbsp;&nbsp;<var-def>
&nbsp;&nbsp;&nbsp;&nbsp;通常输入数据整理、计算、汇总等，对应于清算应用的统计结构体。特别地，内部变量需指定数据源（输入或其它变量）与处理方式（汇总或赋值等）；
#### 1.1.4&nbsp;&nbsp;<constant>
&nbsp;&nbsp;&nbsp;&nbsp;定义报表内使用的常量，例如定义列宽等；&nbsp;

#### 1.1.5&nbsp;&nbsp;<arguments>
&nbsp;&nbsp;&nbsp;&nbsp;打印输出参数：将打印内容变量化，使用arguments申明，报表输出时指定相应参数值进行打印输出，这部分变量与用于计算的变量无关，仅用于报表显示。

### 1.2&nbsp;&nbsp;变量使用方式
常量与报表实例化对象直接使用 ${fieldName} 表示；&nbsp;

内部变量通常在绑定到报表块时使用，需先申明局部变量名，使用 ${变量名.fieldName} 表示，例如:&nbsp;

先声明： var=”rec” binding=”C603DzRec” 
表示方式：${rec.settleAt}打印输出参数使用 #{argName} 表示，例如：&nbsp;

先申明： arguments="transDesc"
指定值： <arg name="transDesc" value="ATM Chargeback" />表示方式：#{ transDesc }  实际输出"ATM Chargeback"

特殊字段数据类型：&nbsp;

amt_n/amt_h: 普通精度、高精度金额，金额字段必须指定对应的币种，用于计算或格式化输出时的转换&nbsp;

### 1.3&nbsp;&nbsp;统计计算
可定义变量之间的统计计算逻辑，例如：&nbsp;

```
<var-def name="C001DFIssRec" src="C001DFInput" alg="sum">&nbsp;

        <field name="rcvSettleCurrCd"  />
        <field name="issSettleAt" />
        <field name="issNetAt"  />
    </var-def>
```
定义内部变量类型C001DFIssRec，包含3个字段，字段类型同输入对象C001DFInput，字段值由输入对象C001DFInput累加。&nbsp;

## 2.&nbsp;&nbsp;报表格式
### 2.1&nbsp;&nbsp;报表行row
```
<row>
		<col width=”50” align=”left”>text</col>
	    <col width=”30”>value</col>
</row>

可以预定义行数据与样式，供其它行引用，例如：
<row-def name=”row=”>
		<col width=”100” fill=”=” />
	</row-def>
<row ref=”row=” />

可以在使用时声明，供其它行使用，例如：
<row definition=”row_head”>
		<col width=”50” align=”left”>text1:</col>
	    <col width=”30” align=”left”>value1</col>
	</row>
<row ref=”row_head” />
		<col>text2</col>
	    <col>value2</col>
</row>
```
### 2.2&nbsp;&nbsp;报表块div
#### 2.2.1&nbsp;&nbsp;流水报表块
普通报表块直接将变量（通常是输入）与报表打印方式进行绑定，在产生变量时进行大打印。例如：
```
<div var="rec" binding="C007DZDetailInput">
        <row ref="row_detail">
            <col>${rec.priAcctNo}</col>
            <col>${rec.transAt}</col>
            <col>${rec.transmsnDtTm}</col>
            <col>${rec.authIdRespCd}</col>
            <col>${rec.voucherNo}</col>
            <col print="amt_n">${rec.mchntFeeAt}</col>
            <col>${rec.termId}</col>
            <col>${rec.retriRefNo}</col>
        </row>
    </div>
```
表示将明细C007DZDetailInput以定义的格式进行输出，用于“流水”类报表。&nbsp; 
#### 2.2.2&nbsp;&nbsp;预定义/汇总报表块
可对变量的输出预先定义，使用时进行引用，通常用于包含多个同类型变量的报表打印，可用于分类统计类报表。&nbsp;

例如：预定义报表块，将输入C603DzInput汇总到C603DZRec，同时指定使用时需要设置参数”transDesc”：&nbsp;
```
<div-def name="div_rec" var="rec" binding="C603DZRec" src="C603DzInput" arguments="transDesc">
        <row ref="row_rec">
            <col>#{transDesc}</col>
            <col defVal="0">${rec.settleNum}</col>
            <col print="amt_n">${rec.transAt}</col>
            <col print="amt_n">${rec.inteDivided}</col>
            <col print="amt_n">${rec.changeFee}</col>
            <col print="amt_n">${rec.netSettAt}</col>
        </row>
  </div-def>

预定义报表块，对C003DZRec进行二次汇总：
<div-def name="div_sum" var="rec" binding="C003DZRec" src="C003DZRec" arguments="transDesc" >
        <row-def ref="row_rec">
            <col>#{transDesc}</col>
            <col>${rec.settleNum}</col>
            <col print="amt_n">${rec.transAt}</col>
            <col print="amt_n">${rec.inteDivided}</col>
            <col print="amt_n">${rec.changeFee}</col>
            <col print="amt_n">${rec.netSettAt}</col>
        </row-def>
    </div-def>
```
使用方式如下：&nbsp;

根据输入进行汇总，使用ref对预定义块进行引用，可设置输入参数的过滤规则<filter>以及代码块所需参数”transDesc”的值：&nbsp; 
```
<div ref="div_rec" varId="1" >            
        <arg name="transDesc" value="ATM Disbursement" />
        <filter>
            <field name="transId" compare="in" value="Ex1,Ex2" />
            <field name="connTp"  compare="ne" value="yy" />
        </filter>
    </div>
	<div ref="div_rec" varId="2">            
        <arg name="transDesc" value=" ATM Inquiry" />
        <filter>
            <or>
                <field name="transId" compare="in" value="Ex3,Ex4" />
                <and>
                    <field name="transId" compare="eq"      value="Ey" />
                    <field name="connTp"  compare="notin"   value="yy" />
                </and>
            </or>
        </filter>
    </div>
```
二次汇总，src指定汇总源，即变量id为1与2的变量：
```
<div ref="div_sum" varId="3" src="varId:1,2" >            
        <arg name="transDesc" value="正常交易小记" />            
    </div>
```


## 3.&nbsp;&nbsp;代码生成与使用
工具根据配置文件生成代码，包括报表对象类、输入与内部变量与，其中变量需要实现汇总计算逻辑。&nbsp; 
### 3.1&nbsp;&nbsp;代码生成
#### 3.1.1&nbsp;&nbsp;报表信息样例(C001DFInf.java)
```
public class C001DFInf extends AbstractReportInfo {
	private String fileDir;//报表所在目录

	private String fileNmDt;//文件名中的日期

	private String insIdCd;//机构代码

	private Date reportDt;//报表日期

	private Date transSettleDt;//交易结算日期

	private Date settleDt;//清算日期

	private String transCurrCd;//交易币种

	private String settleCurrCd;//清算币种

	public String getFileDir() {
		return fileDir;
	}

	public void setFileDir(String fileDir) {
		this.fileDir = fileDir;
	}

	public String getFileNmDt() {
		return fileNmDt;
	}

	public void setFileNmDt(String fileNmDt) {
		this.fileNmDt = fileNmDt;
	}

	public String getInsIdCd() {
		return insIdCd;
	}

	public void setInsIdCd(String insIdCd) {
		this.insIdCd = insIdCd;
	}

	public Date getReportDt() {
		return reportDt;
	}

	public void setReportDt(Date reportDt) {
		this.reportDt = reportDt;
	}

	public Date getTransSettleDt() {
		return transSettleDt;
	}

	public void setTransSettleDt(Date transSettleDt) {
		this.transSettleDt = transSettleDt;
	}

	public Date getSettleDt() {
		return settleDt;
	}

	public void setSettleDt(Date settleDt) {
		this.settleDt = settleDt;
	}

	public String getTransCurrCd() {
		return transCurrCd;
	}

	public void setTransCurrCd(String transCurrCd) {
		this.transCurrCd = transCurrCd;
	}

	public String getSettleCurrCd() {
		return settleCurrCd;
	}

	public void setSettleCurrCd(String settleCurrCd) {
		this.settleCurrCd = settleCurrCd;
	}

}
```
#### 3.1.2&nbsp;&nbsp;汇总中间记录样例(C001DFRec.java)
```
public class C001DFRec {

    private Integer                             settleNum = 0;

    private Double                              transAt = 0.0;

    private Double                              acqSettleAt = 0.0;

    private Double                              rmbFee = 0.0;

    private TreeMap<String, List<C001DFIssRec>> issRecMap = new TreeMap<String, List<C001DFIssRec>>();

    public void addC001DFInput(C001DFInput item) {
        this.settleNum += item.getSettleNum();
        this.acqSettleAt = this.acqSettleAt + item.getAcqSettleAt();
        this.rmbFee = this.rmbFee + item.getRmbFee();

        String issRecMapKey = item.getRcvSettleCurrCd();
        List<C001DFIssRec> issRecMapVal = issRecMap.get(issRecMapKey);
        if (issRecMapVal == null) {
            issRecMapVal = new ArrayList<C001DFIssRec>();
            C001DFIssRec c001DFIssRec = new C001DFIssRec();
            c001DFIssRec.addC001DFInput(item);
            issRecMapVal.add(c001DFIssRec);
            issRecMap.put(issRecMapKey, issRecMapVal);
        } 
    }

    public void addC001DFRec(C001DFRec item) {
        this.settleNum += item.getSettleNum();
        this.acqSettleAt = this.acqSettleAt + item.getAcqSettleAt();
        this.rmbFee = this.rmbFee + item.getRmbFee();
    }

    public Integer getSettleNum() {
        return settleNum;
    }

    public void setSettleNum(Integer settleNum) {
        this.settleNum = settleNum;
    }

    public Double getTransAt() {
        return transAt;
    }

    public void setTransAt(Double transAt) {
        this.transAt = transAt;
    }

    public Double getAcqSettleAt() {
        return acqSettleAt;
    }

    public void setAcqSettleAt(Double acqSettleAt) {
        this.acqSettleAt = acqSettleAt;
    }

    public Double getRmbFee() {
        return rmbFee;
    }

    public void setRmbFee(Double rmbFee) {
        this.rmbFee = rmbFee;
    }

    public Map getIssRecMap() {
        return issRecMap;
    }
}
```
### 3.2&nbsp;&nbsp;汇总逻辑
#### 3.2.1&nbsp;&nbsp;报表工厂(ReportFactory.java)
```
/**
 * 报表生成工厂
 * @author tianyin
 * @version 
 * @since  
 * 
 */
public class ReportFactory {
    
    /**
     * @since 
     * @param xmlPath
     * @param repInfo
     * @return
     */
    public static Report create(String xmlPath, AbstractReportInfo repInfo) {
        return new DefaultReport(xmlPath, repInfo);
    }
    
    /**
     * 创建流水类报表
     * @since 
     * @param xmlPath
     * @param repInfo
     * @return
     */
    public static Report transReport(String xmlPath, AbstractReportInfo repInfo) {
        return new TransReport(xmlPath, repInfo);
    }
    
    /**
     * 创建汇总类报表
     * @since 
     * @param xmlPath
     * @param repInfo
     * @return
     */
    public static Report sumReport(String xmlPath, AbstractReportInfo repInfo) {
        return new SumReport(xmlPath, repInfo);
    }
}
```
#### 3.2.2&nbsp;&nbsp;报表接口(Report.java)
```
/**
 * 报表的解析/生成接口
 * @author tianyin
 * @version 
 * @since  
 * 
 */
public interface Report {
   
    /**
     * 
     * @since 
     * @param xmlPath
     */
    void init(String xmlPath, AbstractReportInfo repInf) throws Exception ;
      
    /**
     * 整个报表头部信息
     * @since 
     * @param rptInfo
     */
    void buildRptHeader(AbstractReportInfo rptInfo) throws Exception;
    
   
    
    /**
     * 按块输出代码格式
     * @since 
     * @param input
     * @param divDef
     */
    void put(Object input, String divName) throws Exception;
    
    /**
     * 缓存def
     * @since 
     * @param defName
     * @param def
     */
    void cachedDef(String defName, Def def);
    
    /**
     * @since 
     * @param defName
     * @return
     */
    Def getCachedDef(String defName);
    /**
     * 获取读写工具
     * @since 
     * @return
     */
    PrintWriter getWriter();
    
    /**
     * 返回报表详情信息
     * @since 
     * @return
     */
    RptFile getRptFile(); 
    
    /**
     * 报表生成
     * @since 
     */
    void flush() throws Exception;
     
    /**
     * 报表文档最终关闭
     * @since 
     */
    void close();
}
```

#### 3.2.3&nbsp;&nbsp;报表抽象接口(AbstractReport.java)
```
/**
 * 默认实现 抽象report类,该类中实现xml的解析
 * @author tianyin
 * @version 
 * @since  
 * 
 */
public abstract class AbstractReport implements Report {
    
    protected String xmlConfigPath                              = null;
    
    //生成的报表文件路径
    protected String rptPath                                    = null;
    
    protected String rptName                                    = null;
        
    //输出到文本
    protected PrintWriter bw                                    = null;
    
    protected  AbstractReportInfo reportInf                     = null;
    
    //xml中预定义的各类def缓存
    protected Map<String, Def>  defMap                          = new HashMap<>();
    
    //xml中预定义的中间变量缓存
    protected Map<String, List<VarDefDesc> >   varDefMap        = new HashMap<>();
    
    //xml解析器
    protected ReportXmlParser xmlParser                         = null;
    
    protected List<Displayable> displays                        = null;

    //当前的显示控件
    protected int currDisplayIdx                                = 0 ;
    
    //判断是否初始化
    protected boolean inited                                    = false;
    
    /** 初始化，并生成相应的头部信息
     * @throws IOException 
     *
     * @see com.cup.report.Report#init(String)
     */
    @Override
    public final void init(String xmlPath, AbstractReportInfo reportInf) throws Exception {
        xmlConfigPath = xmlPath;
        this.reportInf = reportInf;
        rptPath  = reportInf.getAbstractRptPath();
        rptName  = reportInf.getAbstractFileName();
        inited = true;
        
        try {
            //解析xml配置文件
            xmlParser = new ReportXmlParser(xmlConfigPath, this);
            displays = xmlParser.parseXml();
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException("解析报表配置xml时失败");
        }
        
        if (displays == null) {
            throw new Exception("xml中没有定义row或div等显示控件");
        }
        
        try {
            //生成报表文件
            StringBuilder tmpPath= new StringBuilder(rptPath);
            if (rptPath != null && !rptPath.endsWith("//")) {
                tmpPath.append("//");
            }
            bw = new PrintWriter(new FileWriter(tmpPath.append(rptName).toString()));
            
            /**生成头部内容*/
            buildRptHeader(reportInf);
            bw.flush();
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException("初始化时生成报表文件头部失败");
        }
    }
    
   
    
    /**
     * 缓存所有的Def类型
     * @since 
     * @param defName
     * @param def
     */
    @Override
    public  void cachedDef(String defName, Def def) {
        defName = StringUtils.trim(defName);
        if (defName != null) {
            defMap.put(defName, def); 
        }
        
        //缓存中间变量，
        if (def instanceof VarDefDesc) {
            if (!StringUtils.isBlank(((VarDefDesc) def).getSrc())) {
                String[] srcs = StringUtils.split(((VarDefDesc) def).getSrc(), ",");
                for (String s : srcs) {
                    s = StringUtils.trim(s);
                    if (s == null) {
                        return;
                    }
                    if (!varDefMap.containsKey(s)) {
                        List<VarDefDesc> defs = new ArrayList<>();
                        defs.add((VarDefDesc) def);
                        varDefMap.put(s, defs);
                    } else {
                        varDefMap.get(s).add((VarDefDesc) def);
                    }
                }
            }
        }
    }
    
    /**
     * 
     * @since 
     * @param 根据名称获取缓存类
     * @return
     */
    @Override
    public Def getCachedDef(String defName) {
        defName = StringUtils.trim(defName);
        if (defName != null) {
            return defMap.get(defName);
        }
        return null;
    }
     
    /** (non-Javadoc)
     * @throws Exception 
     * @see com.cup.report.Report#put(com.cup.report.AbstractInput)
     */
    @Override
    public void put(Object input, String divName) throws Exception {
        throw new Exception("子类需实现父类的print方法");
    }
    
    /** 获取读写器
     * @see com.cup.report.Report#getWriter()
     */
    @Override
    public PrintWriter getWriter() {
        return bw;
    }

    @Override
    public RptFile getRptFile() {
        return new RptFile(rptPath, rptName, 0);
    }

  //生成报表中的row
    protected void genRow(RowDesc display) throws IllegalAccessException, InvocationTargetException, NoSuchMethodException {
       if (!StringUtils.isBlank(display.getRef())) {//如果row的def不为空
           RowDefDesc rowDef = (RowDefDesc) getCachedDef(display.getRef());
           if (display.getRef().equals("row-")) {
               bw.println("----------------------------------------------------------------------------------------------------------------------------------------------------------------------------");
           } else if (display.getRef().equals("row=")) {
               bw.println("============================================================================================================================================================================");
           } else if (display.getCols() != null) {
               int colSize = display.getCols().size();
               for (int i = 0; i < colSize; i++) {
                   ColDesc col = display.getCols().get(i);
                   ColDesc colDef = rowDef.getCol().get(i);
                   
                   //如果列里面有参数，则用实例中的实际值替代参数
                   String seg = col.getCol();
                   seg = paramReplace(reportInf, seg);
                   
                   bw.print(colAddFixedSpace(seg, colDef));
               }
               bw.println();
           }
       } else {//如果row的def为空
           if (display.getCols() != null) {
               for (ColDesc col : display.getCols()) {
                   //参数化处理
                   String seg = col.getCol();
                   seg = paramReplace(reportInf, seg);
                   bw.print(colAddFixedSpace(seg, col));
               }
               bw.println();
           }
           
       }
    }
    
    //参数替换，xml中含参数的全部替换成报表内容
    protected  String paramReplace(Object input, String segMent) throws IllegalAccessException, InvocationTargetException, NoSuchMethodException {
        String param = PrintCheckUtils.segmentMatch(segMent);
        if ( param != null) {
            String t = BeanUtils.getProperty(input, param);
            segMent = StringUtils.replace(segMent, "${" + param + "}", t);
        }
        return segMent;
    }
    
    /**
     * 给列字符串加上指定空格
     * @since 
     * @param str
     * @param col
     * @return
     */
    protected String colAddFixedSpace(String str, ColDesc col) {
        String align = col.getAlign();
        int width = StringUtils.isBlank(col.getWidth()) ? 0 : Integer.valueOf(col.getWidth());
        return fixedSpaceStrGen(str, align, width);
    }
    
    /**
     * 补齐固定空格的函数
     * 
     * @since
     * @param str
     * @param align
     * @param width
     * @return
     */
    protected String fixedSpaceStrGen(Object str, String align, int width) {
        //空字符串也支持
        if (str instanceof String && StringUtils.isBlank((String)str)) {
            str = "";
        }
        
        String tmp = (String) str;
        StringBuilder sb= new StringBuilder();
        StringBuilder space = new StringBuilder();
        int validLen = width - MyStringUtils.length(tmp);
        for (int i = 0; i < validLen; i++) {
            space.append(" ");
        }
         
        if (align != null && align.equals("right")) {// 除非指定是右对齐，否则默认左对齐
            sb.append(space).append(tmp);
        } else {
            sb.append(tmp).append(space);
        }
        //System.out.println("打印数据:[" + sb.toString() + "]" + " align/width:" + align + "/" + width);
        return sb.toString();
    }
    

    @Override
    public void flush() throws Exception {
        if (bw != null) {
            bw.flush();
        }
    }

    /**强制关闭文件输出 
     * (non-Javadoc)
     * @see com.cup.report.Report#close()
     */
    @Override
    public void close() {
        
        if (bw != null) {
            bw.close();
        }
    }
    
    protected boolean isInited() {
        return inited;
    }
    
}
```
#### 3.2.4&nbsp;&nbsp;报表汇总类(SumReport.java)
```
/**
 * 汇总类报表
 * @author tianyin
 * @version 
 * @since  
 * 
 */
public class SumReport extends AbstractReport {
    
    // 缓存每个div的中间变量, 内容为varId:C001DFRec,如 "1":new C001DFRec()
    Map<String, Object>  divMap     = new HashMap<>();

    Map<String, Boolean> isDivPrint = new HashMap<>();
    
    public static long startTime  = 0L;
    
    SumReport(String xmlPath, AbstractReportInfo reportInf) {
        try {
            init(xmlPath, reportInf);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    /** (non-Javadoc)
     * @see com.cup.report.Report#buildRptHeader(com.cup.report.AbstractReportInfo)
     */
    @Override
    public void buildRptHeader(AbstractReportInfo rptInfo) throws Exception {
    }
    
    /** (non-Javadoc)
     * @throws Exception 
     * @see com.cup.report.Report#put(com.cup.report.AbstractInput)
     */
    @Override
    public void put(Object input, String divName) throws Exception {
        // 往所有的var-def中写数据
        String clazz = ClassUtils.getClassName(input);
        for (Displayable def : displays) {
            // 根据div找到匹配的div-def文件
            if (isRefMatchDivDef(def, clazz)) {
                sumInputWithFilter((DivDesc) def, "add" + clazz, input);
            }
        }
    }
    
    //判断是不是符合匹配的div-def
    private boolean isRefMatchDivDef(Displayable dpl, String src) throws Exception {
    	
    	if (!(dpl instanceof DivDesc)) {
    		return false;
    	}
    	
    	DivDesc tmp = (DivDesc) dpl;
    	
    	if (!defMap.containsKey(tmp.getRef())) {
            throw new Exception(tmp.getRef() + "找不到匹配的块定义");
        }
    	
    	//div的src源要匹配
    	if (StringUtils.isBlank(tmp.getRef()) || !src.equals(((DivDefDesc)defMap.get(tmp.getRef())).getSrc())) {
    		return false;
    	}
    	
    	//添加一条中间汇总实例
    	if (!divMap.containsKey(tmp.getVarId())) {
            divMap.put(tmp.getVarId(), ClassUtils.newInstance("com.cup.gen.c001." + ((DivDefDesc)defMap.get(tmp.getRef())).getBinding()));    	
    	}
    	
    	return true;
    }
    
    /**
     * 带过滤器的汇总计算
     * @param div
     * @param input
     * @throws InvocationTargetException 
     * @throws IllegalArgumentException 
     * @throws IllegalAccessException 
     * @throws NoSuchMethodException 
     */
    private void sumInputWithFilter(DivDesc div, String methodName, Object input) throws Exception {
    	//如果div中含有filter，且filter包含当前记录，则过滤该条记录
        if (!isFiltered(div, input)) {
    	    return ;
    	}
        
        //获取C001DFRec的所有声明方法
    	Object obj = divMap.get(div.getVarId());
    	Method[] methods = obj.getClass().getDeclaredMethods();
    	for (Method method : methods) {
    		if (method.getName().equals(methodName)) {
    			method.invoke(obj, input);
//    			if (obj instanceof C001DFRec) {
//    			    System.out.println("当前div详细:" + div.getVarId());
//    			    System.out.println("当前汇总详细:" + ((C001DFRec)obj).getRmbFee());
//    			}
    		}
    	}
    }
    
    //注意事项，filter中所有字段当字符串处理的，判断div是否有过滤器筛选，写filter时注意即可。
    private boolean isFiltered(DivDesc div, Object input) throws Exception {
        long t1 = System.currentTimeMillis();
        if (div.getFilter() != null && !StringUtils.isEmpty(div.getFilter().getFilterScript())) {
            String script = div.getFilter().getFilterScript();
            script = script.replaceAll("or", "||");
            script = script.replaceAll("and", "&&");
            List<String> args = RegularUtils.allMatchedSegments(script);
            if (args != null) {
                for (String arg : args) {
                    String argValue = BeanUtils.getProperty(input, arg);
                    script = script.replace("${" + arg + "}",  "\""+ argValue + "\"");
                }
            }
            ScriptEngineManager manager = new ScriptEngineManager();
            ScriptEngine engine = manager.getEngineByName("js");        
            Object result = null;
            try {
                result = engine.eval(script);//"(\"No.2.\" == \"Ex1\" || \"No.2.\" == \"Ex2\") || (\"No.2.\" == \"Ey\" && \"0.1\" != \"yy\")");
                startTime += ((System.currentTimeMillis()) - t1 );
                System.out.println("耗时累计:" + startTime/1000.0);
                return (boolean) result;
            } catch (ScriptException e) {
               e.printStackTrace();
               throw e;
            }
    
        }
        return true;
    }
    
    private void genSumDiv(DivDesc div) throws Exception {
    	
    	if (StringUtils.isBlank(div.getRef())) {
    		throw new Exception("div没有定义格式, 不输出");
    	}
    	
    	if (StringUtils.isBlank(div.getFatherId())) {
    	    throw new Exception("div没有定义父块号，不输出");
    	}
    	
    	  DivDefDesc divDef = (DivDefDesc) defMap.get(div.getRef());
          if (divDef == null || divDef.getRows() == null) {
              throw new Exception( div.getRef() +" 对应的div-def不存在或其row不存在");
          }
          
    	//包括汇总和二次汇总
    	SumupDivWithFormatter(div, divDef);
    }
    
    
    private void SumupDivWithFormatter(DivDesc div, DivDefDesc divDef) throws Exception {
        String argumentNm = divDef.getArguments();
        String argumentVal = div.getArg().getValue();

        for (RowDesc row : divDef.getRows()) {
            RowDefDesc rowDef = (RowDefDesc) defMap.get(row.getRef());
            if (rowDef.getCol() != null) {
                if (!isDivPrint.containsKey(div.getFatherId())) {
                    for (ColDesc col : rowDef.getCol()) {
                        bw.print(colAddFixedSpace(col.getCol(), col));
                    }
                    bw.println();
                    isDivPrint.put(div.getFatherId(), true);
                }
          
                Object obj = divMap.get(div.getVarId());
                if (div.getSrc() != null) {//如果是二次汇总,则先把对象二次汇总处理
                    obj = secondSumupRecords(divDef, div, obj);
                }

                int len = rowDef.getCol().size();
                for (int i = 0; i < len; i++) {
                    ColDesc col1 = rowDef.getCol().get(i);
                    ColDesc col2 = row.getCols().get(i);

                    if (col2.getCol().equals("#{" + argumentNm + "}")) {
                        bw.print(colAddFixedSpace(argumentVal, col1));
                    } else {
                        String seg = col2.getCol();
                        //查看是不是需要匹配
                        String param = PrintCheckUtils.segmentMatch(seg);
                        
                        if (param != null) {
                            seg = getRecDetail(obj, seg, param);
                        }
                        bw.print(colAddFixedSpace(seg, col1));
                    }
                }
                bw.println();

            }
        }
        //处理forEach标签
        if (divDef.getForEach() != null) {
            ForEachDesc forEach = divDef.getForEach();
           
            String var  = forEach.getVar();
            String items = forEach.getItems();
            int begin  = StringUtils.isBlank(forEach.getBegin()) ? 0: Integer.valueOf(forEach.getBegin());
            
            RowDesc row = forEach.getRow();
            if (row != null && row.getCols() != null && !StringUtils.isBlank(row.getRef())) {
                int colLen = row.getCols().size();
                RowDefDesc rowDef = (RowDefDesc) defMap.get(row.getRef());
                for (int i = 0; i < colLen; i++) {
                    ColDesc col1 = row.getCols().get(i);
                    ColDesc col2 = rowDef.getCol().get(i);
                    if (!StringUtils.isBlank(col1.getCol())) {
                        col1.setCol("");
                    }
                    System.out.println(col1.getCol());
                    //bw.print(colAddFixedSpace(col1.getCol(), col2));
                }
                //bw.println();
            }
        }
    }
    
   
    /**
     * @since 
     * @param divDef 块格式定义
     * @param div 输出块
     * @param obj 当前块的统计记录
     * @throws Exception
     */
    private Object secondSumupRecords(DivDefDesc divDef, DivDesc div, Object obj) throws Exception {
        if (obj == null) {
            divMap.put(div.getVarId(),
                    ClassUtils.newInstance("com.cup.gen.c001." + ((DivDefDesc) defMap.get(div.getRef())).getBinding()));
        }
        obj = divMap.get(div.getVarId());
        String[] srcs = null;
        if (div.getSrc().startsWith("varId:")) {
            srcs = StringUtils.split(div.getSrc().substring(6), ",");
        } else {
            srcs = StringUtils.split(div.getSrc(), ",");
        }

        for (String src : srcs) {
            Object recTmp = divMap.get(StringUtils.trim(src));
            if (recTmp == null || obj == null) {
                continue;
            }
            String method = "add" + divDef.getSrc();

            Method[] methods = obj.getClass().getDeclaredMethods();
            for (Method m : methods) {
                if (m.getName().equals(method)) {
                    m.invoke(obj, recTmp);
                    // if (obj instanceof C001DFRec) {
                    // System.out.println("当前二次汇总div varId:" + div.getVarId() + "来源：" + src);
                    // System.out.println("当前汇总详细增量:" + ((C001DFRec) recTmp).getRmbFee());
                    // System.out.println("当前汇总后:" + ((C001DFRec) obj).getRmbFee());
                    // }
                }
            }
        }
        
        return obj;
    }
    
    
    
    //获取字段中定义的值
    private String getRecDetail(Object input, String segMent, String param) throws IllegalAccessException,
            InvocationTargetException, NoSuchMethodException {
        String[] paramArr = StringUtils.split(param, ".");
        String t = BeanUtils.getProperty(input, paramArr[1]);
        segMent = StringUtils.replace(segMent, "${" + param + "}", t);
        return segMent;
    }
    
    @Override
    public void flush() throws Exception {
        //完成汇总的编写
    	try {
    		 for (int i = currDisplayIdx; i < displays.size(); i++) {
    	        	Displayable dpl = displays.get(i);
    	        	if (dpl instanceof RowDesc) {
    	        		genRow((RowDesc) dpl);
    	        	} else if (dpl instanceof DivDesc){
    					genSumDiv((DivDesc) dpl);
    				}
    	        }
    	} catch (Exception e) {
    		System.out.println("汇总报表flush条件不满足,未能完成报表生成");
    		e.printStackTrace();
    		throw e;
    	}
        super.flush();
    }
}
```
#### 3.2.5&nbsp;&nbsp;配置xml解析(ReportXmlParser.java)
```
/**
 * 从上到下逐行读取xml配置, 解析报表，序列化输出控件。
 * @author tianyin
 * @version 
 * @since  
 * 
 */
public class ReportXmlParser {
    
    //读取的xml的当前行数
    private int currentRowNo             = 0;
    
    private String xmlPath               = null;
    
    private Report report                = null;
           
    private ReportXmlParser() {
        
    }
    
    /**
     * @param xmlReader
     * @param report
     */
    public ReportXmlParser(String xmlPath, Report report) {
        this.xmlPath = xmlPath;
        this.report    = report; 
    }
    
    /**
     * 或许下一个用于显示的控件，目前只支持row和div控件显示
     * @since 
     * @param xmlPath
     * @param report
     * @return
     * @throws IOException 
     */
    public List<Displayable> parseXml() throws Exception {
        
        List<Displayable> res = new ArrayList<>();
        DocumentBuilderFactory a = DocumentBuilderFactory.newInstance();  
        try {  
            DocumentBuilder b = a.newDocumentBuilder();  
            Document document = b.parse(xmlPath);  
            NodeList nodes = document.getElementsByTagName("report");
            
            //顶级report节点
            Node root = nodes.item(0);
            NodeList childNodes = root.getChildNodes();
            int childNodesLen =  childNodes.getLength();
            for (int i = 0; i < childNodesLen; i++) {
                Node node = childNodes.item(i);
                if (node.getNodeType() != Node.ELEMENT_NODE) {
                    continue;
                }
                if (node.getNodeName().endsWith("def")) {
                    parseDef((Element) node);
                } else if (node.getNodeName().equals("row") || node.getNodeName().equals("div")) {
                    Displayable result = parseDisplay((Element) node);
                    res.add(result);
                }
            }
        } catch (ParserConfigurationException e) {  
            e.printStackTrace(); 
            throw e;
        } catch (SAXException e) {  
            e.printStackTrace();
            throw e;
        } catch (IOException e) {  
            e.printStackTrace();
            throw e;
        }  
        
        return res.isEmpty() ? null : res;
    }
    
    //解析得到Def的组件
    private void parseDef(Element node) throws Exception {
        if (StringUtils.equals(node.getNodeName(), "row-def")) {
            nullThrowEx(node.getAttribute("name"), node.toString() + "的name属性不能为空");

            RowDefDesc rowDef = new RowDefDesc();            
            String name = node.getAttribute("name");
            rowDef.setName(name);
         
            List<ColDesc> colList = parseCols(node);
            rowDef.setCol(colList);
            
            report.cachedDef(name, rowDef);
        } else if (StringUtils.equals(node.getNodeName(), "div-def")) {
            DivDefDesc divDef = new DivDefDesc();
            String name = node.getAttribute("name");
            divDef.setName(name);
            divDef.setSrc(node.getAttribute("src"));
            divDef.setBinding(node.getAttribute("binding"));
            divDef.setVar(node.getAttribute("var"));
            divDef.setArguments(node.getAttribute("arguments"));
            
            //解析for-each和rows
            NodeList rowNodes = node.getChildNodes();
            int len = rowNodes.getLength();
            List<RowDesc> rowList = new ArrayList<>();
            for (int i = 0; i < len; i++) {
                Node e =  rowNodes.item(i);
                if (e.getNodeType() != Node.ELEMENT_NODE) {
                    continue;
                }
                if ("row".equals(e.getNodeName())) {
                    RowDesc row = buildRow((Element) e);
                    rowList.add(row);
                } else if ("forEach".equals(e.getNodeName())) {
                    ForEachDesc forEach = buildForEach((Element) e);
                    divDef.setForEach(forEach);
                }
            }
            divDef.setRows(rowList);
            report.cachedDef(name, divDef);
        } else if (StringUtils.equals(node.getNodeName(), "var-def")) {
            //TODO 如果是var-def解析
            VarDefDesc varDef = new VarDefDesc();
            String name = node.getAttribute("name");
            varDef.setName(name);
            varDef.setAlg(node.getAttribute("alg"));
            varDef.setSrc(node.getAttribute("src"));
            
            //解析method和Field标签
            NodeList childNodes = node.getChildNodes();
            List<FieldDesc> fieldList = new ArrayList<>();
            List<MethodDesc> methodList  = new ArrayList<>();
            for (int i =0 ; i < childNodes.getLength(); i++) {
                Node e = childNodes.item(i);
                if (e.getNodeType() != Node.ELEMENT_NODE) {
                    continue;
                }
                if ("field".equals(e.getNodeName())) {
                    FieldDesc field = buildField((Element) e);
                    fieldList.add(field);
                } else if ("method".equals(e.getNodeName())) {
                     MethodDesc method = buildMethodDesc((Element) e);
                     methodList.add(method);
                    }
            }
            varDef.setFields(fieldList);
            varDef.setMethods(methodList);
            
            report.cachedDef(name, varDef);
        }
        
    }
   
    //解析得到显示组件
    private Displayable parseDisplay(Element node) throws Exception {
        if (StringUtils.equals(node.getNodeName(), "row")) {
            return buildRow(node);
        } else if (StringUtils.equals(node.getNodeName(), "div")) {
            nullThrowEx(node.getAttribute("varId"), "存在未设置varId的div元素,请检查");
            nullThrowEx(node.getAttribute("fatherId"), "存在未设置fatherId的div元素,请检查");
            
            DivDesc div = new DivDesc();
            div.setRef(node.getAttribute("ref"));
            div.setBinding(node.getAttribute("binding"));
            div.setVarId(node.getAttribute("varId"));
            div.setSrc(node.getAttribute("src"));
            div.setFatherId(node.getAttribute("fatherId"));

            NodeList childNodes = node.getChildNodes();
            int childLen = childNodes.getLength();
            List<RowDesc> rowList = new ArrayList<>();
            for (int i = 0; i < childLen; i++) {
                Node e =  childNodes.item(i);
                if (e.getNodeType() != Node.ELEMENT_NODE) {
                    continue;
                }
                if ("arg".equals(e.getNodeName())) {
                    ArgDesc arg = new ArgDesc();
                    arg.setName(((Element) e).getAttribute("name"));
                    arg.setValue(((Element) e).getAttribute("value"));
                    div.setArg(arg);
                } else if ("row".equals(e.getNodeName())) {
                    rowList.add(buildRow((Element) e));
                } else if ("filter".equals(e.getNodeName())) {
                    FilterDesc filter = new FilterDesc();
                    filter.setName(((Element) e).getAttribute("name"));
                    filter.setFilterScript(e.getTextContent());
                    div.setFilter(filter);
                }
            }
            if (!rowList.isEmpty()) {
                div.setRows(rowList);
            }
            return div;
        }
        return null;
    }
    
    private RowDesc buildRow(Element node) {
        RowDesc row = new RowDesc();
        row.setRef(node.getAttribute("ref"));
        row.setDefinition(node.getAttribute("definition"));
        
        List<ColDesc> colList = parseCols(node);
        row.setCols(colList);
        
        return row;
    }
    
    //解析生成col列
    private List<ColDesc> parseCols(Element node) {
        NodeList childNodes = node.getChildNodes();
        int childLen = childNodes.getLength();
        List<ColDesc> colList = new ArrayList<>();
        for (int i = 0; i < childLen; i++) {
            Node e =  childNodes.item(i);
            if (e.getNodeType() != Node.ELEMENT_NODE) {
                continue;
            }
            if (e.getNodeName().equals("col")) {
                ColDesc col = new ColDesc();
                col.setCol(e.getTextContent());
                col.setAlign(((Element) e).getAttribute("align"));
                col.setFill(((Element) e).getAttribute("fill"));
                col.setPrint(((Element) e).getAttribute("print"));
                col.setSpan(((Element) e).getAttribute("span"));
                col.setWidth(((Element) e).getAttribute("width")); 
                colList.add(col);
            }             
        }
        
        return colList.isEmpty() ? null : colList;
    }
    
  //field解析
    private FieldDesc buildField(Element ele) throws Exception {
        nullThrowEx(ele.getAttribute("type"), "存在未指定type类型的field元素,请检查");
        
        FieldDesc field = new FieldDesc();
        field.setCompare(ele.getAttribute("compare"));
        field.setCurrCd(ele.getAttribute("currCd"));
        field.setDesc(ele.getAttribute("desc"));
        field.setLength(ele.getAttribute("length"));
        field.setName(ele.getAttribute("name"));
        field.setSort(ele.getAttribute("sort"));
        field.setSrc(ele.getAttribute("src"));
        field.setType(ele.getAttribute("type"));
        field.setValue(ele.getAttribute("value"));
        return field;
    }
    
    //div中的method解析
    private MethodDesc buildMethodDesc(Element ele) throws Exception {
        nullThrowEx(ele.getAttribute("name"), "存在未指定name属性的method元素,请检查");

        MethodDesc method = new MethodDesc();
        method.setCode(ele.getAttribute("code"));
        method.setName(ele.getAttribute("name"));
        method.setType(ele.getAttribute("type"));
        
        return method;
    }
    //div中forEach只支持一个row
    private ForEachDesc buildForEach(Element ele) {
        ForEachDesc forEach = new ForEachDesc();
        forEach.setVar(ele.getAttribute("var"));
        forEach.setItems(ele.getAttribute("item"));
        forEach.setBegin(ele.getAttribute("begin"));
        
        NodeList nodes = ele.getChildNodes();
        for (int i = 0; i < nodes.getLength(); i++) {
            Node e = nodes.item(i);
            if (e.getNodeType() != Node.ELEMENT_NODE) {
                continue;
            }
            if (e.getNodeName().equals("row")) {
                RowDesc row = buildRow((Element) e); 
                forEach.setRow(row);
                break;
            }
        }
        return forEach;
    }

    /**
     * 返回当前行号
     * @since 
     * @return
     */
    public int getCurrentNo() {
        return currentRowNo;
    }

    /**
     * 为空则抛出含msg消息恶的Exception
     * @since 
     * @param str
     * @param msg
     * @throws Exception
     */
    private void nullThrowEx(String str, String msg) throws Exception {
        if (StringUtils.isBlank(str)) {
            throw new Exception(msg);
        }
    }
}
```
## 4.&nbsp;&nbsp;总结
&nbsp;&nbsp;&nbsp;&nbsp;略
## 5.&nbsp;&nbsp;作者介绍
&nbsp;&nbsp;&nbsp;&nbsp;国际业务团队

