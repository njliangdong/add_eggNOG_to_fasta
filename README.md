#!/usr/bin/env python3
import sys
import argparse

# 定义注释字段到列索引的映射（0-indexed）
FIELD_INDEX = {
    "Description": 7,  # 对应第8列
    "PFAMs": 20        # 对应第21列
}

def load_annotation_mapping(eggnog_file, field):
    """
    读取 eggNOG 注释文件，提取基因 ID 和指定字段的注释内容。
    参数:
      eggnog_file: eggNOG 输出文件（制表符分隔）
      field: 指定要提取的字段名称，例如 "PFAMs" 或 "Description"
    """
    mapping = {}
    # 判断输入的注释字段是否存在
    if field not in FIELD_INDEX:
        print("没有这个eggNOG项目")
        sys.exit(1)
    col_index = FIELD_INDEX[field]
    with open(eggnog_file) as f:
        for line in f:
            # 跳过注释或空行
            if line.startswith("#") or not line.strip():
                continue
            fields = line.strip().split("\t")
            if len(fields) <= col_index:
                continue
            gene_id = fields[0]
            annotation = fields[col_index]
            mapping[gene_id] = annotation
    return mapping

def process_fasta(input_fasta, mapping, output_fasta):
    """
    读取 FASTA 文件，处理每个序列头：
      1. 清除原头部中“>”后第一个空格之后的所有内容，仅保留基因 ID；
      2. 如果 mapping 中存在该基因的注释，则在基因 ID 后添加一个空格和注释信息；
      3. 序列其他部分保持不变。
    """
    with open(input_fasta) as fin, open(output_fasta, "w") as fout:
        for line in fin:
            if line.startswith(">"):
                # 仅保留基因ID（即“>”后第一个空格之前的内容）
                gene_id = line[1:].strip().split()[0]
                new_header = ">" + gene_id
                if gene_id in mapping and mapping[gene_id]:
                    new_header += " " + mapping[gene_id]
                fout.write(new_header + "\n")
            else:
                fout.write(line)

def main():
    parser = argparse.ArgumentParser(description="将 eggNOG 注释添加到 FASTA 序列头中。")
    parser.add_argument("-input_fasta", required=True, help="输入的 FASTA 文件")
    parser.add_argument("-eggNOG_table", required=True, help="eggNOG 注释文件（制表符分隔）")
    parser.add_argument("-annotation_categories", required=True,
                        help="指定添加的注释字段，如 'PFAMs' 或 'Description'")
    parser.add_argument("-output_fasta", required=True, help="输出的 FASTA 文件")
    
    args = parser.parse_args()
    
    mapping = load_annotation_mapping(args.eggNOG_table, args.annotation_categories)
    process_fasta(args.input_fasta, mapping, args.output_fasta)
    print(f"{args.annotation_categories} 注释已添加到 FASTA 文件，输出保存为：{args.output_fasta}")

if __name__ == "__main__":
    main()

