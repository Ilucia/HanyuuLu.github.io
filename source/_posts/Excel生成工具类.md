### 一次性excel导出

```java
    public void exportProductExcel() throws IOException {
        File file = new File("product.txt");
        StringBuilder resultString = new StringBuilder();
        try{
            BufferedReader br = new BufferedReader(new FileReader(file));//构造一个BufferedReader类来读取文件
            String s = null;
            while((s = br.readLine())!=null){//使用readLine方法，一次读一行
                resultString.append(System.lineSeparator()+s);
            }
            br.close();
        }catch(Exception e){
            e.printStackTrace();
        }

        JSONObject result = JSONObject.parseObject(resultString.toString());
        System.out.println(resultString);
        JSONArray productList = result.getJSONArray("data");
        HSSFWorkbook workbook = new HSSFWorkbook();
        //创建excel文档
        HSSFSheet sheet = workbook.createSheet("商品表");

        //定义列的宽度
        sheet.setColumnWidth(0, 20 * 256);
        sheet.setColumnWidth(1, 15 * 256);
        sheet.setColumnWidth(2, 20 * 256);
        sheet.setColumnWidth(3, 10 * 256);
        sheet.setColumnWidth(4, 10 * 256);
        sheet.setColumnWidth(5, 10 * 256);
        sheet.setColumnWidth(6, 20 * 256);
        sheet.setColumnWidth(7, 30 * 256);
        sheet.setColumnWidth(8, 30 * 256);
        //设置表头
        HSSFRow headerRow = sheet.createRow(0);
        headerRow.createCell(0).setCellValue("商品编码");
        headerRow.createCell(1).setCellValue("外部商品编码");
        headerRow.createCell(2).setCellValue("商品名称");
        headerRow.createCell(3).setCellValue("大类");
        headerRow.createCell(4).setCellValue("中类");
        headerRow.createCell(5).setCellValue("小类");
        headerRow.createCell(6).setCellValue("品牌");
        headerRow.createCell(7).setCellValue("售卖机型");
        headerRow.createCell(8).setCellValue("标签");

        for (int i = 0;i < productList.size();i++){
            JSONObject product = productList.getJSONObject(i);
            HSSFRow row = sheet.createRow(i+1);
            row.createCell(0).setCellValue(product.getString("skuCode"));
            row.createCell(1).setCellValue(product.getString("externalSkuId"));
            row.createCell(2).setCellValue(product.getString("name"));
            row.createCell(3).setCellValue(product.getString("bigLevelSpuName"));
            row.createCell(4).setCellValue(product.getString("mediumLevelSpuName"));
            row.createCell(5).setCellValue(product.getString("productSpuName"));
            row.createCell(6).setCellValue(product.getString("product_brand_cn"));
            row.createCell(7).setCellValue(product.getString("machine_type_names"));
            JSONArray productTagList = product.getJSONArray("productTagVOList");
            StringBuilder tags = new StringBuilder();
            if(productTagList != null && !productTagList.isEmpty()){
                for(int j = 0;j < productTagList.size();j++) {
                    JSONObject tag = productTagList.getJSONObject(j);
                    tags.append(" ").append(tag.getString("tagName"));
                }
            }
            if(tags.length() > 0){
                row.createCell(8).setCellValue(tags.substring(1));
            }else{
                row.createCell(8).setCellValue("");
            }
        }

        //将excel写入到ByteArrayOutStream中
        FileOutputStream out = new FileOutputStream("商品列表.xls");
        workbook.write(out);
    }
```

### excel导出

```java
    public void exportProductExcel() throws IOException {
        PageResultVO result = productSkuController.getProductSkuByPage(null, null, null, null, null, null, 1, 900, null);

        List<VemProductSkuResultMap> productList = (List<VemProductSkuResultMap>) result.getData();
        HSSFWorkbook workbook = new HSSFWorkbook();
        //创建excel文档
        HSSFSheet sheet = workbook.createSheet("商品表");

        //定义列的宽度
        sheet.setColumnWidth(0, 5 * 256);
        sheet.setColumnWidth(1, 5 * 256);
        sheet.setColumnWidth(2, 10 * 256);
        sheet.setColumnWidth(3, 10 * 256);
        sheet.setColumnWidth(4, 10 * 256);
        sheet.setColumnWidth(5, 10 * 256);
        sheet.setColumnWidth(6, 10 * 256);
        sheet.setColumnWidth(7, 10 * 256);
        sheet.setColumnWidth(8, 10 * 256);
        //设置表头
        HSSFRow headerRow = sheet.createRow(0);
        headerRow.createCell(0).setCellValue("商品编码");
        headerRow.createCell(1).setCellValue("外部商品编码");
        headerRow.createCell(2).setCellValue("商品名称");
        headerRow.createCell(3).setCellValue("大类");
        headerRow.createCell(4).setCellValue("中类");
        headerRow.createCell(5).setCellValue("小类");
        headerRow.createCell(6).setCellValue("品牌");
        headerRow.createCell(7).setCellValue("售卖机型");
        headerRow.createCell(8).setCellValue("标签");

        for (int i = 0;i < productList.size();i++){
            VemProductSkuResultMap product = productList.get(i);
            HSSFRow row = sheet.createRow(i+1);
            row.createCell(0).setCellValue(product.getId());
            row.createCell(1).setCellValue(product.getExternalSkuId());
            row.createCell(2).setCellValue(product.getName());
            row.createCell(3).setCellValue(product.getBigLevelSpuName());
            row.createCell(4).setCellValue(product.getMediumLevelSpuName());
            row.createCell(5).setCellValue(product.getProductSpuName());
            String cn = DictConsts.UNMATCH_VALUE;
            SkuBrandFacade skuBrandFacade = SpringContextUtil.getBean("skuBrandFacade");
            VemSkuBrand skuBrand = skuBrandFacade.getSkuBrand(product.getBrandCode());
            if(!Objects.isNull(skuBrand)){
                cn = skuBrand.getName();
            }
            row.createCell(6).setCellValue(cn);
            List<VemProductSkuMachineType> skuMachineTypes = product.getMachineTypeList();
            StringBuilder typeNames = new StringBuilder();
            if(CollectionUtils.isNotEmpty(skuMachineTypes)){
                DictService dictService= SpringContextUtil.getBean("dictServiceImpl");
                List<VemDict> dictList=dictService.listVemDictByDictGroupId(DictConsts.MACHINE_TYPE);
                if(CollectionUtils.isNotEmpty(dictList)){
                    for(VemDict dict : dictList){
                        String value = String.valueOf(dict.getDictCode());
                        for(VemProductSkuMachineType type : skuMachineTypes){
                            if(value.equals(type.getMachineType())){
                                typeNames.append(",").append(dict.getDictName());
                                break;
                            }
                        }
                    }
                }
            }
            if(typeNames.length() > 0){
                row.createCell(7).setCellValue(typeNames.substring(1));
            }else{
                row.createCell(7).setCellValue(DictConsts.UNMATCH_VALUE);
            }

            List<ProductTagVO> productTagList = product.getProductTagVOList();
            StringBuilder tags = new StringBuilder();
            if(CollectionUtils.isNotEmpty(productTagList)){
                for (ProductTagVO tag : productTagList) {
                    tags.append(" ").append(tag.getTagName());
                }
            }
            if(tags.length() > 0){
                row.createCell(8).setCellValue(tags.substring(1));
            }else{
                row.createCell(8).setCellValue("");
            }
        }

        //将excel写入到ByteArrayOutStream中
        FileOutputStream out = new FileOutputStream("商品列表.xls");
        workbook.write(out);
    }
```



```java
{"商品编码", "商品Id", "商品名称", "商品别名", "外部商品编码", "市场指导价(元)", "类目", "品牌", "建议售价(元)", "计量单位", "规格", "厚度(mm)", "重量(g)", "图片", "保质期(天)", "售卖机型", "标签", "是否原材料", "是否组合商品", "是否返信用", "是否非专卖", "批次来源", "定时折扣", "折扣模板", "修改时间", "修改人"}
```



```java
package com.nfsq.vem.admin.entity.dto;

import lombok.Getter;
import lombok.Setter;

import java.math.BigDecimal;
import java.util.Date;

/**
 * @author niu
 * @date 2020/11/24 下午5:13
 */
@Setter
@Getter
public class ProductExcelModel {
    private String skuCode;
    private Integer id;
    private String name;
    private String nameAlias;
    private String externalSkuId;
    private BigDecimal marketPrice;
    private String productSpuLevelName;
    private String productBrand;
    private BigDecimal guidePrice;
    private String measurementUnit;
    private String specification;
    private Integer thickness;
    private Integer weight;
    private String imageUrl;
    private Integer guaranteePeriod;
    private String machineTypeNames;
    private String tags;
    private String description;
    private String isOrigin;
    private String isGroupSkuCnName;
    private String isReturnCreditCnName;
    private String ifNonMonopolyCnName;
    private String ifBatchNoCnName;
    private Date updateDate;
    private String updatePerson;
}

```





```java
/**
 * 导出商品品项列表
 *
 * @param productName
 * @return
 */
@ResponseBody
@RequestMapping(value = "/exportProductSkuExcel", method = RequestMethod.GET)
public void exportProductExcel(@RequestParam(required = false) String brandCode,
                               @RequestParam(required = false) String spuId,
                               @RequestParam(required = false) String productName,
                               @RequestParam(required = false) String nameAlias,
                               @RequestParam(required = false) Integer tagId,
                               @RequestParam(required = false) String machineType,
                               @RequestParam(required = false) Boolean isMaterialProduct,
                               HttpServletResponse response) throws IOException {
    PageResultVO result = getProductSkuByPage(brandCode, spuId, productName, nameAlias, tagId, machineType, null, null, isMaterialProduct);
    List<VemProductSkuResultMap> productList = (List<VemProductSkuResultMap>) result.getData();

    PoiExcelExport poiExcelExport = new PoiExcelExport(response, "商品品项", "商品品项");
    // 列名
    String[] columnNames = {"商品编码", "商品Id", "商品名称", "商品别名", "外部商品编码", "市场指导价(元)", "类目", "品牌", "建议售价(元)", "计量单位", "规格", "厚度(mm)", "重量(g)", "图片", "保质期(天)", "售卖机型", "标签", "描述", "是否原材料", "是否组合商品", "是否返信用", "是否非专卖", "批次来源", "修改时间", "修改人"};
    // map中的key
    String[] keys = { "skuCode", "id", "name", "nameAlias", "externalSkuId",
            "marketPrice", "productSpuLevelName", "productBrand", "guidePrice", "measurementUnit",
            "specification", "thickness", "weight", "imageUrl", "guaranteePeriod",
            "machineTypeNames", "tags", "description", "isOrigin", "isGroupSkuCnName",
            "isReturnCreditCnName", "ifNonMonopolyCnName", "ifBatchNoCnName", "updateDate", "updatePerson"};
    int[] titleSize = { 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15};

    List<ProductExcelModel> models = constructProductExcelModel(productList);
    poiExcelExport.writeExcelBaseMethod("商品品项", keys, columnNames, titleSize, models);
    poiExcelExport.writeExcelMultiSheet();
}

private List<ProductExcelModel> constructProductExcelModel(List<VemProductSkuResultMap> productSkuResultMaps) {
    List<ProductExcelModel> models = new ArrayList<>();
    for (VemProductSkuResultMap product : productSkuResultMaps) {
        ProductExcelModel model = new ProductExcelModel();
        model.setSkuCode(product.getSkuCode());
        model.setId(product.getId());
        model.setName(product.getName());
        model.setNameAlias(product.getNameAlias());
        model.setExternalSkuId(product.getExternalSkuId());
        model.setMarketPrice(product.getMarketPrice());
        model.setProductSpuLevelName(product.getProductSpuLevelName());
        String cn = DictConsts.UNMATCH_VALUE;
        SkuBrandFacade skuBrandFacade = SpringContextUtil.getBean("skuBrandFacade");
        VemSkuBrand skuBrand = skuBrandFacade.getSkuBrand(product.getBrandCode());
        if(!Objects.isNull(skuBrand)){
            cn = skuBrand.getName();
        }
        model.setProductBrand(cn);
        model.setGuidePrice(product.getGuidePrice());
        model.setMeasurementUnit(product.getMeasurementUnit());
        model.setSpecification(product.getSpecification());
        model.setThickness(product.getThickness());
        model.setWeight(product.getWeight());
        model.setImageUrl(product.getImageUrl());
        model.setGuaranteePeriod(product.getGuaranteePeriod());
        List<VemProductSkuMachineType> skuMachineTypes = product.getMachineTypeList();
        StringBuilder typeNames = new StringBuilder();
        if(CollectionUtils.isNotEmpty(skuMachineTypes)){
            DictService dictService= SpringContextUtil.getBean("dictServiceImpl");
            List<VemDict> dictList=dictService.listVemDictByDictGroupId(DictConsts.MACHINE_TYPE);
            if(CollectionUtils.isNotEmpty(dictList)){
                for(VemDict dict : dictList){
                    String value = String.valueOf(dict.getDictCode());
                    for(VemProductSkuMachineType type : skuMachineTypes){
                        if(value.equals(type.getMachineType())){
                            typeNames.append(",").append(dict.getDictName());
                            break;
                        }
                    }
                }
            }
        }
        if(typeNames.length() > 0){
            model.setMachineTypeNames(typeNames.substring(1));
        }else{
            model.setMachineTypeNames(DictConsts.UNMATCH_VALUE);
        }
        List<ProductTagVO> productTagList = product.getProductTagVOList();
        StringBuilder tags = new StringBuilder();
        if(CollectionUtils.isNotEmpty(productTagList)){
            for (ProductTagVO tag : productTagList) {
                tags.append(" ").append(tag.getTagName());
            }
        }
        if(tags.length() > 0){
            model.setTags(tags.substring(1));
        }else{
            model.setTags("");
        }
        model.setDescription(product.getDescription());
        model.setIsOrigin(("2".equals(product.getProductType())) ? "是" : "否");
        model.setIsGroupSkuCnName(product.getIsGroupSkuCnName());
        model.setIsReturnCreditCnName(product.getIsReturnCreditCnName());
        model.setIfNonMonopolyCnName(product.getIfNonMonopolyCnName());
        model.setIfBatchNoCnName(product.getIfBatchNoCnName());
        model.setUpdateDate(product.getUpdateDate());
        model.setUpdatePerson(product.getUpdatePerson());
        models.add(model);
    }
    return models;
}
```



```java
@Override
public List<VemProductSkuResultMap> listVemProductSkuResultMapByPage(List<String> spuIds, String brandCode,String productSkuName, String nameAlias, boolean isMaterialProduct, String machineType, Integer pageNum, Integer pageSize) {
    if (pageNum != null && pageSize != null) {
        PageHelper.startPage(pageNum,pageSize);
    }
    List<VemProductSkuResultMap> vemProductSkuList = manualVemProductSkuMapper.selectVemProductSkuResultMapBySkuNameAndSpuIds(null, productSkuName, nameAlias, spuIds, brandCode,isMaterialProduct,machineType);
    return vemProductSkuList;
}
```

```sql
ALTER TABLE vem_machine ADD WAREHOUSE_ADDRESS CHAR(1) AFTER STATUS;
```



```java

    /**
     * 导出商品品项列表
     *
     * @param productName
     * @return
     */
    @ResponseBody
    @RequestMapping(value = "/exportProductSkuExcel", method = RequestMethod.GET)
    public void exportProductExcel(@RequestParam(required = false) String brandCode,
                                   @RequestParam(required = false) String spuId,
                                   @RequestParam(required = false) String productName,
                                   @RequestParam(required = false) String nameAlias,
                                   @RequestParam(required = false) Integer tagId,
                                   @RequestParam(required = false) String machineType,
                                   @RequestParam(required = false) Boolean isMaterialProduct,
                                   HttpServletResponse response) {
        PageResultVO result = getProductSkuByPage(brandCode, spuId, productName, nameAlias, tagId, machineType, null, null, isMaterialProduct);
        List<VemProductSkuResultMap> productList = (List<VemProductSkuResultMap>) result.getData();

        PoiExcelExport poiExcelExport = new PoiExcelExport(response, "商品品项", "商品品项");
        // 列名
        String[] columnNames = {"商品编码", "商品Id", "商品名称", "商品别名", "外部商品编码",
                "市场指导价(元)", "类目", "品牌", "建议售价(元)", "计量单位",
                "规格", "厚度(mm)", "重量(g)", "图片", "保质期(天)",
                "售卖机型", "标签", "描述", "是否原材料", "是否组合商品",
                "是否返信用", "是否非专卖", "批次来源", "修改时间", "修改人"};
        // map中的key
        String[] keys = { "skuCode", "id", "name", "nameAlias", "externalSkuId",
                "marketPrice", "productSpuLevelName", "productBrand", "guidePrice", "measurementUnit",
                "specification", "thickness", "weight", "imageUrl", "guaranteePeriod",
                "machineTypeNames", "tags", "description", "isOrigin", "isGroupSkuCnName",
                "isReturnCreditCnName", "ifNonMonopolyCnName", "ifBatchNoCnName", "updateDate", "updatePerson"};
        int[] titleSize = { 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15};

        List<ProductExcelModel> models = constructProductExcelModel(productList);
        poiExcelExport.writeExcelBaseMethod("商品品项", keys, columnNames, titleSize, models);
        poiExcelExport.writeExcelMultiSheet();
    }

    private List<ProductExcelModel> constructProductExcelModel(List<VemProductSkuResultMap> productSkuResultMaps) {
        List<ProductExcelModel> models = new ArrayList<>();
        SkuBrandFacade skuBrandFacade = SpringContextUtil.getBean("skuBrandFacade");
        Map<String, String> brandMap = skuBrandFacade.findSkuBrands();
        DictService dictService= SpringContextUtil.getBean("dictServiceImpl");
        List<VemDict> dictList=dictService.listVemDictByDictGroupId(DictConsts.MACHINE_TYPE);
        for (VemProductSkuResultMap product : productSkuResultMaps) {
            ProductExcelModel model = new ProductExcelModel();
            model.setSkuCode(product.getSkuCode());
            model.setId(product.getId());
            model.setName(product.getName());
            model.setNameAlias(product.getNameAlias());
            model.setExternalSkuId(product.getExternalSkuId());
            model.setMarketPrice(product.getMarketPrice());
            model.setProductSpuLevelName(product.getProductSpuLevelName());
            String cn = DictConsts.UNMATCH_VALUE;
            String skuBrand = brandMap.get(product.getBrandCode());
            if(StringUtil.isNotBlank(skuBrand)) {
                cn = skuBrand;
            }
            model.setProductBrand(cn);
            model.setGuidePrice(product.getGuidePrice());
            model.setMeasurementUnit(product.getMeasurementUnit());
            model.setSpecification(product.getSpecification());
            model.setThickness(product.getThickness());
            model.setWeight(product.getWeight());
            model.setImageUrl(product.getImageUrl());
            model.setGuaranteePeriod(product.getGuaranteePeriod());
            List<VemProductSkuMachineType> skuMachineTypes = product.getMachineTypeList();
            StringBuilder typeNames = new StringBuilder();
            if(CollectionUtils.isNotEmpty(skuMachineTypes)){
                if(CollectionUtils.isNotEmpty(dictList)){
                    for(VemDict dict : dictList){
                        String value = String.valueOf(dict.getDictCode());
                        for(VemProductSkuMachineType type : skuMachineTypes){
                            if(value.equals(type.getMachineType())){
                                typeNames.append(",").append(dict.getDictName());
                                break;
                            }
                        }
                    }
                }
            }
            if(typeNames.length() > 0){
                model.setMachineTypeNames(typeNames.substring(1));
            }else{
                model.setMachineTypeNames(DictConsts.UNMATCH_VALUE);
            }
            List<ProductTagVO> productTagList = product.getProductTagVOList();
            StringBuilder tags = new StringBuilder();
            if(CollectionUtils.isNotEmpty(productTagList)){
                for (ProductTagVO tag : productTagList) {
                    tags.append(" ").append(tag.getTagName());
                }
            }
            if(tags.length() > 0){
                model.setTags(tags.substring(1));
            }else{
                model.setTags("");
            }
            model.setDescription(product.getDescription());
            model.setIsOrigin(("2".equals(product.getProductType())) ? "是" : "否");
            model.setIsGroupSkuCnName(product.getIsGroupSkuCnName());
            model.setIsReturnCreditCnName(product.getIsReturnCreditCnName());
            model.setIfNonMonopolyCnName(product.getIfNonMonopolyCnName());
            model.setIfBatchNoCnName(product.getIfBatchNoCnName());
            model.setUpdateDate(product.getUpdateDate());
            model.setUpdatePerson(product.getUpdatePerson());
            models.add(model);
        }
        return models;
    }
```

##### 提交信息

```
【Fix】ID1107326 商品信息导出功能】 https://www.tapd.cn/52268405/prong/stories/view/1152268405001107326   1.导出文件并下载初步完成
```

