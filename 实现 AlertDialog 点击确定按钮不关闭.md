# 实现 AlertDialog 点击确定按钮不关闭 #
当我们在使用AlertDialog时，无论点击“确定”(PositiveButton)还是“取消”(NegativeButton)，对话框都会消失，这在某些情景下是不合理的。例如下面这种情况：AlertDialog 里面有必填或必选项，当用户没有输入或选择时就点击了确定，这时对话框不应该消失的，而是应该提示用户完成填写或选择后才能进行下一步。

关于 AlertDialog 实现上面的效果，百度了一下多是自定义 “确定”(PositiveButton)和“取消”(NegativeButton) 按钮 或通过反射的方式来实现（例如这篇：http://blog.csdn.net/notonlyforshe/article/details/7955323 ），这里介绍一个简单的方法来实现这个效果。

通过看 AlertDialog 的源码，我们可以发现，真正控制 AlertDialog 的三大按钮逻辑都在 AlertController.java 中。在 AlertController 里，自动为每个按钮添加了一个 OnclickListener 监听，代码如下：

    private void setupButtons(ViewGroup buttonPanel) {
        ...
        mButtonPositive.setOnClickListener(mButtonHandler);
		...
        mButtonNegative.setOnClickListener(mButtonHandler);
		...
        mButtonNeutral.setOnClickListener(mButtonHandler);
		...
    }

下面我们看一下 mButtonHandler 是怎么实现的：

    private final View.OnClickListener mButtonHandler = new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            final Message m;
            if (v == mButtonPositive && mButtonPositiveMessage != null) {
                m = Message.obtain(mButtonPositiveMessage);
            } else if (v == mButtonNegative && mButtonNegativeMessage != null) {
                m = Message.obtain(mButtonNegativeMessage);
            } else if (v == mButtonNeutral && mButtonNeutralMessage != null) {
                m = Message.obtain(mButtonNeutralMessage);
            } else {
                m = null;
            }

            if (m != null) {
                m.sendToTarget();
            }

            // Post a message so we dismiss after the above handlers are executed
            mHandler.obtainMessage(ButtonHandler.MSG_DISMISS_DIALOG, mDialog)
                    .sendToTarget();
        }
    };

可以看出无论点击那个按钮都会执行 

	mHandler.obtainMessage(ButtonHandler.MSG_DISMISS_DIALOG, mDialog)
                    .sendToTarget();

而这句会调用 AlertDialog 的 dismiss 方法，所以只要点击按钮就会关闭 AlertDialog ；那么解决的方法也就有了：在 setupButtons 调用之后，确定按钮被点击之前重新为确定按钮设置一个 OnClickListener ，这样就不会发送上面的消息，也就不会关闭 AlertDialog 了；为了简化逻辑，我们可以在 AlertDialog 显示后再设置该 OnClickListener 。

考虑到代码的复用性，我们最好把确定按钮的 OnClickListener 和 AlertDialog 的 DialogInterface.OnClickListener 整合为一个类，使用时根据参数来判断是否在点击确定按钮后关闭 AlertDialog 。例如，我们在展示一个 AlertDialog 时封装一个静态方法，如下：

    /**
     * @param context
     * @param title
     * @param view
     * @param onClickListener
     * @param dimissable      点击 确定 按钮后弹框是否消失， 
	 *     true 消失 ， false 不消失，不消失时应该主动关闭弹框
     */
    public static void displayAlertDialogTwoButton(Context context, String title, View view, CustomDialogInterface.OnClickListener onClickListener, boolean dimissable) {
        AlertDialog.Builder builder = new AlertDialog.Builder(context)
                .setTitle(title)
                .setView(view)
                .setCancelable(true)
                .setPositiveButton("确定", dimissable ? onClickListener : null)
                .setNegativeButton("取消", onClickListener);
        alertDialog = builder.create();
        alertDialog.show();
        if (!dimissable) {
            onClickListener.setDialog(alertDialog);
            alertDialog.getButton(DialogInterface.BUTTON_POSITIVE).setOnClickListener(onClickListener);
        }
    }

如上面那样根据一个参数 dimissable 来控制确定按钮的行为需要我们自定义一个 DialogInterface.OnClickListener , 代码如下：

	public abstract class CustomDialogInterface implements DialogInterface {

    public static abstract class OnClickListener implements  View.OnClickListener ， DialogInterface.OnClickListener {

        private DialogInterface dialog;
        protected final int SUBMIT = DialogInterface.BUTTON_POSITIVE;
        protected final int CANCLE = DialogInterface.BUTTON_NEGATIVE;

        public OnClickListener() {
        }

        /**
         * 该构造方法用于点击 确定 按钮 ，弹框不消失的需求，一般使用上面无参构造方法即可
         *
         * @param dialog
         */
        public OnClickListener(DialogInterface dialog) {
            this.dialog = dialog;
        }
		
        @Override
        public final void onClick(View v) {
            super.onClick(v);
            if (null != dialog && dialog instanceof AlertDialog) {
                if (v == ((AlertDialog) dialog).getButton(BUTTON_POSITIVE)) {
                    onClick(dialog, BUTTON_POSITIVE);
                } else if (v == ((AlertDialog) dialog).getButton(BUTTON_NEGATIVE)) {
                    onClick(dialog, BUTTON_NEGATIVE);
                } else if (v == ((AlertDialog) dialog).getButton(BUTTON_NEUTRAL)) { 
					// 回到 AlertDialog 的正常逻辑
                    onClick(dialog, BUTTON_NEUTRAL);
                } else {
                    LogTool.i("该 alertdialog 的点击位置不是三大按钮");
                }
            } else {
                LogTool.i("没有调用 setDialog 或 该 dialog 不是 alertdialog 不能通过一般 View 的 setOnclickListener 来代替 DialogInterface.OnClickListener");
            }
        }

        /**
         * @param dialog
         * @param which
         */
        @Override
        public void onClick(DialogInterface dialog, int which) {

        }

		/**
		 * 当使用无参构造方法时，可以主动为 dialog 赋值
		 */
        public void setDialog(DialogInterface dialog) {
            this.dialog = dialog;
        }
    }

    @Override
    public void cancel() {

    }

    @Override
    public void dismiss() {

    }
	}

**注意**：上面的 public final void onClick(View v) 方法时 final 的不提供覆写，所以为 AlertDialog 添加 CustomDialogInterface 的使用效果和正常逻辑没有区别，最终都会走向 onClick(DialogInterface dialog, int which) 。这样我们就可以根据使用场景为 dimissable 赋值皆可，其它不用关心。