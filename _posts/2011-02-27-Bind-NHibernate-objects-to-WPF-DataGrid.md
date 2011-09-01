---
layout: post
title: Bind NHibernate objects to WPF DataGrid
---

{{ page.title }}
================

Lately I have felt the need to try out Windows Presentation Foundation. I have been using WinForms with Ado.NET providers but it just feels I’m coupling too much business logic with the user interface.

Windows Presentation Foundation seems very promising for implementing the Model-View-ViewModel pattern. Specifically when combined with an ORM, like NHibernate. Even though this post isn’t implementing the pattern properly, it should give you the direction technically :). I will be using NHibernate, Fluent NHibernate and SQLite for the examples.

The problem with WPF and NHibernate is, that there isn’t an out-of-box two-way binding for the components. WinForms and Ado.NET classes can be coupled so that retrieving and persisting the data can be accomplished with just a few mouse clicks in Visual Studio Designer. For WPF and NHibernate you need some more manual work, but personally I think it’s worth it, as the resulting code is much cleaner and all in your own hands.

## Part1: The Models

### Customer

{% highlight csharp %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace FluentNHibernateDataGridMappingTest.Model
{
    public class Customer
    {
        public virtual int Id { get; set; }
        public virtual string Name { get; set; }
        public virtual string Email { get; set; }
        public virtual string Phone { get; set; }

        public virtual IList Addresses { get; set; }

        public Customer()
        {
            Addresses = new List();
        }

        public virtual void AddAddress(CustomerAddress address)
        {
            address.Customer = this;
            Addresses.Add(address);
        }
    }
}
{% endhighlight %}

### CustomerAddress

Actually this class isn’t used in the binding, but it demonstrates nicely the Fluent NHibernate mappings :)

{% highlight csharp %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace FluentNHibernateDataGridMappingTest.Model
{
    public class CustomerAddress
    {
        public virtual int Id { get; set; }
        public virtual string Street { get; set; }
        public virtual string Zip { get; set; }
        public virtual string City { get; set; }
        public virtual string Country { get; set; }

        public virtual Customer Customer { get; set; }
    }
}
{% endhighlight %}

## Part2: The ModelViews

### CustomerView

So basically, the purpose for this class is to implement the INorifyPropertyChanged interface, providing the necessary triggers for two-way binding..

{% highlight csharp %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ComponentModel;

namespace FluentNHibernateDataGridMappingTest.Model
{
    public class CustomerView : INotifyPropertyChanged
    {
        private Customer _customer;

        public Customer InnerCustomer { get { return _customer; } }

        public CustomerView()
        {
            _customer = new Customer();
        }

        public CustomerView(Customer customer)
        {
            _customer = customer;
        }

        public virtual int Id
        {
            get { return _customer.Id; }
            set
            {
                _customer.Id = value;
                NotifyPropertyChanged("Id");
            }
        }
        public virtual string Name
        {
            get { return _customer.Name; }
            set
            {
                _customer.Name = value;
                NotifyPropertyChanged("Name");
            }
        }
        public virtual string Email
        {
            get { return _customer.Email; }
            set
            {
                _customer.Email = value;
                NotifyPropertyChanged("Email");
            }
        }
        public virtual string Phone
        {
            get { return _customer.Phone; }
            set
            {
                _customer.Phone = value;
                NotifyPropertyChanged("Phone");
            }
        }

        #region INotifyPropertyChanged Members

        public event PropertyChangedEventHandler PropertyChanged;

        protected void NotifyPropertyChanged(string propertyChanged)
        {
            if (PropertyChanged != null)
                PropertyChanged(this, new PropertyChangedEventArgs(propertyChanged));
        }

        #endregion
    }
}
{% endhighlight %}

## Part3: The Collection

This is actually maybe the main part of this “tutorial”. You need a collection, that notifies the user interface about modifications in the collection and the items in the collection.

A suitable base is ObservableCollection<T>. Basically, we just bind bind an event handler for the event raised by INotifyPropertyChanged for each item in the collection.

{% highlight csharp %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Collections.ObjectModel;
using System.ComponentModel;
using System.Collections.Specialized;

namespace FluentNHibernateDataGridMappingTest.Model.Collection
{

    class EcObservableCollection<T> : ObservableCollection<T> where T : INotifyPropertyChanged
    {
        public EcObservableCollection() : base()
        {
            this.CollectionChanged += 
				new NotifyCollectionChangedEventHandler(EcObservableCollection_CollectionChanged);		
        }

        void EcObservableCollection_CollectionChanged(object sender, NotifyCollectionChangedEventArgs e)
        {
            if (e.Action == NotifyCollectionChangedAction.Add) {
                foreach (T item in e.NewItems)
                    item.PropertyChanged += new PropertyChangedEventHandler(item_PropertyChanged);
            }
        }
        public EcObservableCollection(List<T> items) : this()
        {
            foreach (T item in items) {
                this.Add(item);
            }
        }

        void item_PropertyChanged(object sender, PropertyChangedEventArgs e)
        {
            EcObservableCollectionItemChangedEventArgs<T> args = 
				new EcObservableCollectionItemChangedEventArgs<T>();
            args.Item = (T)sender;
            ItemChanged(this, args);
        }

        public event EcObservableCollectionItemChangedEventHandler ItemChanged;

        public delegate void EcObservableCollectionItemChangedEventHandler(object sender, 
															EcObservableCollectionItemChangedEventArgs<T> args);
    }

    class EcObservableCollectionItemChangedEventArgs<T> : EventArgs
    {
        public T Item { get; set; }
    }
}
{% endhighlight %}

## Part4: The user interface

### MainWindow.xaml

This code also demonstrates how you can make the row header to show the id of the item.

{% highlight xml %}
<Window x:Class="FluentNHibernateDataGridMappingTest.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MainWindow" Height="504" Width="821" DataContext="{Binding}">
    <Grid>
        <Grid.Resources>
            <Style x:Key="RowHeadersWithId" TargetType="{x:Type DataGridRowHeader}">
                <Setter Property="Content" Value="{Binding Id}" />
            </Style>
        </Grid.Resources>
        <DataGrid AutoGenerateColumns="False" 
					Margin="12" 
					Name="dataGrid1" 
					AlternatingRowBackground="AliceBlue" 
					RowHeaderStyle="{StaticResource RowHeadersWithId}" 
					HeadersVisibility="All" 
					CanUserAddRows="True">
            <DataGrid.Columns>
                <DataGridTextColumn Binding="{Binding Name}" Header="Name" Width="*"/>
                <DataGridTextColumn Binding="{Binding Email}" Header="Email" />
                <DataGridTextColumn Binding="{Binding Phone}" Header="Phone" />
            </DataGrid.Columns>
        </DataGrid>
    </Grid>
</Window>
{% endhighlight %}

## Part5: Make it all work together

### NHibernate mappings

Here we use Fluent NHibernate bindings, you could of course use the traditional mappings, if you would want to.

#### CustomerMap.cs

{% highlight csharp %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using FluentNHibernate.Mapping;

namespace FluentNHibernateDataGridMappingTest.Model.Mapping
{
    public class CustomerMap : ClassMap
    {
        public CustomerMap()
        {
            Id(x => x.Id);
            Map(x => x.Email);
            Map(x => x.Name);
            Map(x => x.Phone);
            HasMany(x => x.Addresses);
        }
    }
}
{% endhighlight %}

#### CustomerAddressMap.cs

{% highlight csharp %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using FluentNHibernate.Mapping;

namespace FluentNHibernateDataGridMappingTest.Model.Mapping
{
    public class CustomerAddressMap : ClassMap
    {
        public CustomerAddressMap()
        {
            Id(x => x.Id);
            Map(x => x.City);
            Map(x => x.Country);
            Map(x => x.Street);
            Map(x => x.Zip);
            References(x => x.Customer);
        }
    }
}
{% endhighlight %}

### Database configuration: The Fluent NHibernate way

{% highlight csharp %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using NHibernate;
using FluentNHibernate.Cfg;
using FluentNHibernate.Cfg.Db;
using NHibernate.Cfg;
using System.IO;
using NHibernate.Tool.hbm2ddl;

namespace FluentNHibernateDataGridMappingTest.Core
{
    class Database
    {
        public static ISessionFactory CreateSessionFactory()
        {
            return Fluently.Configure()
                .Database(SQLiteConfiguration.Standard.UsingFile("test.db"))
                .Mappings(m => m.FluentMappings.AddFromAssemblyOf<App>())
                .ExposeConfiguration(BuildDataBase)
                .BuildSessionFactory();
        }

        private static void BuildDataBase(Configuration conf)
        {
            conf.SetProperty(NHibernate.Cfg.Environment.ShowSql, "true");
            if (!File.Exists("test.db"))
                new SchemaExport(conf).Create(false, true);
        }
    }
}
{% endhighlight %}

### The user interface initialization and data binding: MainWindow.xaml.cs

{% highlight csharp %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Navigation;
using System.Windows.Shapes;
using NHibernate;
using FluentNHibernateDataGridMappingTest.Core;
using FluentNHibernateDataGridMappingTest.Model;
using System.Collections.ObjectModel;
using FluentNHibernateDataGridMappingTest.Model.Collection;
using System.Collections.Specialized;

namespace FluentNHibernateDataGridMappingTest
{
    public partial class MainWindow : Window
    {
        private ISessionFactory sessionFactory;

        private EcObservableCollection<CustomerView> customers;
        public MainWindow()
        {
            InitializeComponent();
            sessionFactory = Database.CreateSessionFactory();
            using (var session = sessionFactory.OpenSession()) {
                var criteria = session.CreateCriteria<Customer>();
                customers = new EcObservableCollection<CustomerView>();
                foreach (Customer c in criteria.List<Customer>())
                    customers.Add(new CustomerView(c));
            }
            customers.CollectionChanged += 
				new NotifyCollectionChangedEventHandler(customers_CollectionChanged);
            customers.ItemChanged += 
				new EcObservableCollection<CustomerView>.EcObservableCollectionItemChangedEventHandler(customers_ItemChanged);
            dataGrid1.ItemsSource = customers;
        }

        void customers_ItemChanged(object sender, EcObservableCollectionItemChangedEventArgs<CustomerView> args)
        {
            using (var session = sessionFactory.OpenSession()) {
                session.SaveOrUpdate(args.Item.InnerCustomer);
                session.Flush();
            }
        }

        void customers_CollectionChanged(object sender, System.Collections.Specialized.NotifyCollectionChangedEventArgs e)
        {
            using (var session = sessionFactory.OpenSession()) {
                switch (e.Action) {
                    case System.Collections.Specialized.NotifyCollectionChangedAction.Remove:
                        foreach (CustomerView c in e.OldItems) {
                            session.Delete(c.InnerCustomer);
                            session.Flush();
                        }
                        break;
                    case System.Collections.Specialized.NotifyCollectionChangedAction.Add:
                        foreach (CustomerView c in e.NewItems)
                            session.Save(c.InnerCustomer);
                        break;
                    default:
                        foreach (CustomerView c in e.OldItems)
                            session.SaveOrUpdate(c.InnerCustomer);
                        break;
                }
            }
                
        }
    }
}
{% endhighlight %}

## Conclusion

So, with my code examples, it should be easy to figure out how to build quickly different views for your models. The observablecollection I created, is highly reusable, as it takes whatever modelviews you create, as long as they implement the INotifyPropertyChanged interface.

In a nutshell, the steps needed to get two-way databinding with WPF and NHibernate to work together:
 * Write your Models with appropriate ModelViews
 * Map them to database
 * Write (or copy, or search with google) an appropriate Collection
 * Use Visual Studio Designer to implement the user interface
 * Add some code to fetch the items from db with NHibernate and attach some event handlers
 * Enjoy.

If you have anything to add or ask, please leave a comment in this blog or contact me directly via email: niklas@narhinen.net