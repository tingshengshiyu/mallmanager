<template>
	<el-card>
		<my-bread level1='商品管理' level2='分类参数'></my-bread>
		</p>
		<el-alert title="只允许为第三级参数设置分类" type="error" :closable="false"></el-alert>
		</p>
		<el-form label-position="left" label-width="80px" style="height:400px;">
			<!-- 级联选择器 -->
			<el-form-item label="商品分类">
				<el-cascader expand-trigger="hover" :options="options" v-model="selectedOptions" :props="defaultProp" @change="handleChange()"></el-cascader>
			</el-form-item>

			<!-- tab -->

			<el-tabs v-model="active" type="card" @tab-click="handleClick">
				<el-tab-pane label="动态参数" name="1">
					<el-button type="danger" plain>设置动态参数</el-button>

					<!-- 展开行 -->
					<el-table :data="arrDyparams" style="width: 100%">
						<el-table-column type="expand" label="#">
							<template slot-scope="scope">

								<!-- el-tag -->
								<el-tag :key="tag" v-for="tag in scope.row.attr_vals" closable :disable-transitions="false" @close="handleClose(scope.row,tag)">
									{{tag}}
								</el-tag>
								<el-input class="input-new-tag" v-if="inputVisible" v-model="inputValue" ref="saveTagInput" size="small"
								 @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)">
								</el-input>
								<el-button v-else class="button-new-tag" size="small" @click="showInput">+ New Tag</el-button>
							</template>
						</el-table-column>
						<el-table-column label="属性名称" prop="attr_name">
						</el-table-column>
						<el-table-column label="操作" prop="desc">
							<el-button size="mini" plain type="primary" icon="el-icon-edit" circle @click='showEditUserDia(scope.row)'></el-button>
							<el-button size="mini" plain type="danger" icon="el-icon-delete" circle @click="showDeleUserMsgBox(scope.row.id)"></el-button>
						</el-table-column>
					</el-table>
				</el-tab-pane>


				<el-tab-pane label="静态参数" name="2">
					<el-button type="danger" plain>设置静态参数</el-button>

					<el-table :data="arrStaticparams" style="width: 100%">
						<el-table-column type="index" label="#">
						</el-table-column>
						<el-table-column label="属性名称" prop="attr_name">
						</el-table-column>
						<el-table-column label="属性值" prop="attr_vals">
						</el-table-column>
						<el-table-column label="操作" prop="desc">
							<el-button size="mini" plain type="primary" icon="el-icon-edit" circle @click='showEditUserDia(scope.row)'></el-button>
							<el-button size="mini" plain type="danger" icon="el-icon-delete" circle @click="showDeleUserMsgBox(scope.row.id)"></el-button>
						</el-table-column>
					</el-table>
				</el-tab-pane>
			</el-tabs>

		</el-form>


	</el-card>
</template>

<script>
	
	export default {
		
		data() {
			return {
				active: '',
				// 级联选择器绑定的数据
				options: [],
				//级联选择器默认展示
				selectedOptions: [],
				//defaultProp把数据转换成options中需要的值
				defaultProp: {
					label: 'cat_name',
					value: 'cat_id',
					children: 'children'
				},
				//动态参数的数据数组
				arrDyparams: [],
				//静态参数的数据数组
				arrStaticparams: [],
				//tag数据
				inputVisible: false,
				inputValue: ''
			}
		},
		created() {
			this.getGoodCate();
		},
		methods: {
			//点击tab切换时
			async handleClick() {
				if (this.active === '2') {
					if (this.selectedOptions.length === 3) {
						
					}
				}
			},
			//获取三级分类的信息
			async getGoodCate() {
				const res = await this.$http.get(`categories?type=3`);
				//console.log(res);
				this.options = res.data.data;
				//console.log(this.options);
			},
			// 级联选择器@change触发的方法
			async handleChange() {
				if (this.selectedOptions.length === 3) {
					//获取数据
					//id->分类 sel=many表示的是获取动态参数的数据
					const res = await this.$http.get('categories/' + this.selectedOptions[2] + '/attributes?sel=many');
					//console.log(res);
					this.arrDyparams = res.data.data;
					//console.log(this.arrDyparams);
					//this.arrDyparams每个对象.attr_vals字符串->数组->v-for
					this.arrDyparams.forEach(item => {
						//并不是所有的三级分类都有动态参数->""->[]->v-for报错
						// if (item.attr_vals.Length !==O) {
						// item.attr_vaLs = item.attr_vals.trim( ).split( ',')
						//}
						//item.attr_vals里的值为字符串，需要转成对象在遍历，trim()去前后空格，split(',')转换成对象以“，”分割
						item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.trim().split(',')
					});
					//console.log(this.selectedOptions);
					//获取静态参数数据
					const res1 = await this.$http.get('categories/' + this.selectedOptions[2] + '/attributes?sel=only');
					//console.log(res);
					this.arrStaticparams = res1.data.data;

				} else {
					if (this.selectedOptions.length !== 3) {
						//提示
						this.$message.error('请先选择商品分类中的三级分类');
						//刷新视图
						this.selectedOptions = [1, 3, 99999];
						const res = await this.$http.get('categories/' + this.selectedOptions[2] + '/attributes?sel=many');
						this.arrDyparams = res.data.data;
						this.arrDyparams.forEach(item => {
							//并不是所有的三级分类都有动态参数->""->[]->v-for报错
							// if (item.attr_vals.Length !==O) {
							// item.attr_vaLs = item.attr_vals.trim( ).split( ',')
							//}
							//item.attr_vals里的值为字符串，需要转成对象在遍历，trim()去前后空格，split(',')转换成对象以“，”分割
							item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.trim().split(',')

						});
						//console.log(this.selectedOptions);
						const res1 = await this.$http.get('categories/' + this.selectedOptions[2] + '/attributes?sel=only');
						this.arrStaticparams = res1.data.data;
						return;
					}
				}
			},
			// tag方法
			//tag 点击x的时候 ->发送请求
			async handleClose(row, tag) {
				row.attr_vals.splice(row.attr_vals.indexOf(tag), 1);
				var attr_name, attr_sel, attr_vals;
				let putData = {
					attr_name: row.attr_name,
					attr_sel: 'many',
					attr_vals: row.attr_vals.join(',')
				}
				//发送请求
				const res = await this.$http.put("categories/" + this.selectedOptions[2] + "/attributes/" + row.attr_id, putData);
				//console.log(res);
			},
			//tag 点击newTag+按钮
			async showInput(row) {
				this.inputVisible = true;
				this.$nextTick(_ => {
					this.$refs.saveTagInput.$refs.input.focus();
				});
			},
			//tag 回车键 or失去焦点
			async handleInputConfirm(row) {
				let inputValue = this.inputValue;
				if (inputValue) {
					row.attr_vals.push(inputValue);
					//发送请求
					var attr_name, attr_sel, attr_vals;
					let putData = {
						attr_name: row.attr_name,
						attr_sel: 'many',
						attr_vals: row.attr_vals.join(',')
					}
					const res = await this.$http.put("categories/" + this.selectedOptions[2] + "/attributes/" + row.attr_id, putData);
					//console.log(res);
				}
				this.inputVisible = false;
				this.inputValue = "";
			}
		}
	}
</script>

<style>
	.el-tag+.el-tag {
		margin-left: 10px;
	}

	.button-new-tag {
		margin-left: 10px;
		height: 32px;
		line-height: 30px;
		padding-top: 0;
		padding-bottom: 0;
	}

	.input-new-tag {
		width: 90px;
		margin-left: 10px;
		vertical-align: bottom;
	}
</style>
