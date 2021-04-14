

pinctrl-names 定义了状态名称列表： default (i2c 功能) 和 gpio 两种状态。
pinctrl-0 定义了状态 0 (即 default）时需要设置的 pinctrl: &i2c4_xfer
pinctrl-1 定义了状态 1 (即 gpio) 时需要设置的 pinctrl: &i2c4_gpio