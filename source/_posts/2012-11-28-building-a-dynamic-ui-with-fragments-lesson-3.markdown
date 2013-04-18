---
layout: post
title: "Android Training 03 - 使用Fragments建立动态的UI[Lesson 4 - Fragment之间的通信]"
date: 2012-11-28 18:33
comments: true
sidebar: false
categories: Android
---

为了重用fragment的UI组件，你必须为每个fragment建立自己的container，模块化自己的layout与行为。一旦你定义了那些可重用的fragment，你可以使用activity与他们建立联系，对那些UI组件做组合动作等。

通常，你也会想要fragment-s之间能够交流。例如，基于用户事件来改变内容。所有的Fragment-to-Fragment之间的交互都是基于activity进行操作的。两个fragment之间没有办法直接交互。

## Define an Interface[定义接口]
为了允许fragment与activity进行交互，你可以在fragment中定义一个interface，然后在activity中去implement它。Fragment在它的onAttach()方法里面捕获接口的实现，然后call接口的方法来与activity进行交互。

<!-- more -->

下面是一个fragment与activity进行交互的例子：
```java
public class HeadlinesFragment extends ListFragment {
    OnHeadlineSelectedListener mCallback;

    // Container Activity must implement this interface
    public interface OnHeadlineSelectedListener {
        public void onArticleSelected(int position);
    }

    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        
        // This makes sure that the container activity has implemented
        // the callback interface. If not, it throws an exception
        try {
            mCallback = (OnHeadlineSelectedListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString()
                    + " must implement OnHeadlineSelectedListener");
        }
    }
    
    ...
}
```
那么现在，fragment可以通过onArticleSelected() 传递messages给activity。这个方法去使用OnHeadlineSelectedListener接口的mCallback

例如：下面在fragment中的方法会在用户点击listitem的时候被call到。Fragment使用这个callback接口来传递事件给父activity。
```java
    @Override
    public void onListItemClick(ListView l, View v, int position, long id) {
        // Send the event to the host activity
        mCallback.onArticleSelected(position);
    }
```

## Implement the Interface[实现接口]
为了接受到fragment返回事件的callback，包含那个fragment的activity必须实现定义在fragment中的接口。

例如：下面的activity实现了上面例子中的接口。
```java
public static class MainActivity extends Activity
        implements HeadlinesFragment.OnHeadlineSelectedListener{
    ...
    
    public void onArticleSelected(int position) {
        // The user selected the headline of an article from the HeadlinesFragment
        // Do something here to display that article
    }
}
```

## Deliver a Message to a Fragment[传递msg给fragment]
宿主activity可以通过findFragmentById()捕获到fragment实例，然后call fragment的public方法来传递msg给fragment。

如下演示了使用另外一个fragment来显示从上面例子的callback中获取到的内容。
```java
public static class MainActivity extends Activity
        implements HeadlinesFragment.OnHeadlineSelectedListener{
    ...

    public void onArticleSelected(int position) {
        // The user selected the headline of an article from the HeadlinesFragment
        // Do something here to display that article

        ArticleFragment articleFrag = (ArticleFragment)
                getSupportFragmentManager().findFragmentById(R.id.article_fragment);

        if (articleFrag != null) {
            // If article frag is available, we're in two-pane layout...

            // Call a method in the ArticleFragment to update its content
            articleFrag.updateArticleView(position);
        } else {
            // Otherwise, we're in the one-pane layout and must swap frags...

            // Create fragment and give it an argument for the selected article
            ArticleFragment newFragment = new ArticleFragment();
            Bundle args = new Bundle();
            args.putInt(ArticleFragment.ARG_POSITION, position);
            newFragment.setArguments(args);
        
            FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();

            // Replace whatever is in the fragment_container view with this fragment,
            // and add the transaction to the back stack so the user can navigate back
            transaction.replace(R.id.fragment_container, newFragment);
            transaction.addToBackStack(null);

            // Commit the transaction
            transaction.commit();
        }
    }
}
```

*********************************
**学习自：[https://developer.android.com/training/basics/fragments/communicating.html](https://developer.android.com/training/basics/fragments/communicating.html)，请多指教，谢谢！**  
**转载请注明出自[http://kesenhoo.github.com](http://kesenhoo.github.com)，谢谢配合！**






