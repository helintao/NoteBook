# 获取LayoutInflater
- **LayoutInflater inflater = getLayoutInflater();**

    在activity中使用。

    源码：
    ```java
    //Activity.java
    /**
     * Convenience for calling
     * {@link android.view.Window#getLayoutInflater}.
     */
    @NonNull
    public LayoutInflater getLayoutInflater() {
        return getWindow().getLayoutInflater();
    }

    //Window.java实现类PhoneWindow.java
    public PhoneWindow(Context context) {
        super(context);
        mLayoutInflater = LayoutInflater.from(context);
    }
    ```

- **LayoutInflater inflater = LayoutInflater.from(this);**
    
    源码：
    ```java
    //LayoutInflater.java
    /**
     * Obtains the LayoutInflater from the given context.
     */
    public static LayoutInflater from(Context context) {
        LayoutInflater LayoutInflater =
                (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        if (LayoutInflater == null) {
            throw new AssertionError("LayoutInflater not found.");
        }
        return LayoutInflater;
    }
    ```

- **LayoutInflater LayoutInflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);**

    这三种实现，默认最终都是调用了最后一种方式。

# LayoutInflater#inflate
其inflater一共有四个重载方法，最终都是调用了最后一种实现。

- **inflate(@LayoutRes int resource, @Nullable ViewGroup root)**

    该方法，接收两个参数，一个是需要加载的xml文件的id，一个是该xml需要添加的布局，根据root的情况，返回值分为两种：

    - 如果root == null，那么返回这个root
    - 如果root != null，那么返回传入xml的根View

    源码
    ```java
    public View inflate(@LayoutRes int resource, @Nullable ViewGroup root) {
        return inflate(resource, root, root != null);
    }
    ```

- **inflate(XmlPullParser parser, @Nullable ViewGroup root)**

    它提供的不是xml的id，而是XmlPullParser，但是由于View的渲染依赖于xml在编译时的预处理，因此，这个方法并不合适。

    源码
    ```java
    public View inflate(XmlPullParser parser, @Nullable ViewGroup root) {
        return inflate(parser, root, root != null);
    }
    ```

- **inflate(@LayoutRes int resource, @Nullable ViewGroup root, booleanattachToRoot)**

    源码
    ```java
    public View inflate(@LayoutRes int resource, @Nullable ViewGroup root,  booleanattachToRoot) {
        final Resources res = getContext().getResources();
        if (DEBUG) {
            Log.d(TAG, "INFLATING from resource: \"" + res.getResourceName(resource) +  "\" ("
                    + Integer.toHexString(resource) + ")");
        }
        final XmlResourceParser parser = res.getLayout(resource);
        try {
            return inflate(parser, root, attachToRoot);
        } finally {
            parser.close();
        }
    }
    ```

- **inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot)**

    源码
    ```java
    public View inflate(XmlPullParser parser, @Nullable ViewGroup root, booleanattachToRoot) {
    synchronized (mConstructorArgs) {
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");

        final Context inflaterContext = mContext;
        final AttributeSet attrs = Xml.asAttributeSet(parser);
        Context lastContext = (Context) mConstructorArgs[0];
        mConstructorArgs[0] = inflaterContext;
        View result = root;

        try {
            // Look for the root node.
            int type;
            while ((type = parser.next()) != XmlPullParser.START_TAG &&
                    type != XmlPullParser.END_DOCUMENT) {
                // Empty
            }

            if (type != XmlPullParser.START_TAG) {
                throw new InflateException(parser.getPositionDescription()
                        + ": No start tag found!");
            }

            final String name = parser.getName();

            if (DEBUG) {
                System.out.println("**************************");
                System.out.println("Creating root view: "
                        + name);
                System.out.println("**************************");
            }

            if (TAG_MERGE.equals(name)) {
                if (root == null || !attachToRoot) {
                    throw new InflateException("<merge /> can be used only with a valid "
                            + "ViewGroup root and attachToRoot=true");
                }

                rInflate(parser, root, inflaterContext, attrs, false);
            } else {
                // Temp is the root view that was found in the xml
                final View temp = createViewFromTag(root, name, inflaterContext, attrs);

                ViewGroup.LayoutParams params = null;

                if (root != null) {
                    if (DEBUG) {
                        System.out.println("Creating params from root: " +
                                root);
                    }
                    // Create layout params that match root, if supplied
                    params = root.generateLayoutParams(attrs);
                    if (!attachToRoot) {
                        // Set the layout params for temp if we are not
                        // attaching. (If we are, we use addView, below)
                        temp.setLayoutParams(params);
                    }
                }

                if (DEBUG) {
                    System.out.println("-----> start inflating children");
                }

                // Inflate all children under temp against its context.
                rInflateChildren(parser, temp, attrs, true);

                if (DEBUG) {
                    System.out.println("-----> done inflating children");
                }

                // We are supposed to attach all the views we found (int temp)
                // to root. Do that now.
                if (root != null && attachToRoot) {
                    root.addView(temp, params);
                }

                // Decide whether to return the root that was passed in or the
                // top view found in xml.
                if (root == null || !attachToRoot) {
                    result = temp;
                }
            }

        } catch (XmlPullParserException e) {
            final InflateException ie = new InflateException(e.getMessage(), e);
            ie.setStackTrace(EMPTY_STACK_TRACE);
            throw ie;
        } catch (Exception e) {
            final InflateException ie = new InflateException(parser.getPositionDescription()
                    + ": " + e.getMessage(), e);
            ie.setStackTrace(EMPTY_STACK_TRACE);
            throw ie;
        } finally {
            // Don't retain static reference on context.
            mConstructorArgs[0] = lastContext;
            mConstructorArgs[1] = null;

            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }

        return result;
    }
    }
    ```

## 总结
- 作用：从特定的xml节点渲染出一个新的view层级。
- 提示：为了性能考虑，不应当在运行时使用XmlPullParser来渲染布局。
- 参数parser：包含有描述xml布局层级的parser xml dom。
- 参数root，可以是渲染的xml的parent（attachToRoot == true），或者仅仅是为了给渲染的xml层级提供LayoutParams。
- 参数attachToRoot：渲染的View层级是否被添加到root中，如果不是，那么仅仅为xml的根布局生成正确的LayoutParams。
- 返回值：如果attachToRoot为真，那么返回root，否则返回渲染的xml的根布局。

