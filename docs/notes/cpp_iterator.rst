========
Iterator
========

.. contents:: Table of Contents
    :backlinks: none

Reverse Range-based for Loop
----------------------------

.. code-block:: cpp

    // via boost
    // $ g++ --std=c++14 -Wall -Werror -g -O3 reverse.cpp
    // $ ./a.out
    // dlrow olleh

    #include <iostream>
    #include <string>
    #include <boost/range/adaptor/reversed.hpp>

    using namespace boost;

    int main(int argc, char *argv[]) {
      std::string in = "hello world";
      std::string out;
      for (const auto &c : adaptors::reverse(in)) {
          out += c;
      }
      std::cout << out << "\n";
    }

Customize an Iterator
---------------------

.. code-block:: cpp

    // $ g++ -std=c++17 -Wall -Werror -g -O3 a.cc

    #include <iostream>
    #include <memory>

    template <typename T>
    class Array
    {
     public:
      class iterator
      {
        public:
          iterator(T *ptr) : ptr_{ptr} {}
          iterator operator++() { auto i = *this; ++ptr_; return i; }
          iterator operator++(int) { ++ptr_; return *this;};
          T &operator*() { return *ptr_; }
          T *operator->() { return ptr_; }
          bool operator==(const iterator &rhs) { return ptr_ == rhs.ptr_; }
          bool operator!=(const iterator &rhs) { return ptr_ != rhs.ptr_; }
        private:
          T *ptr_;
      };

      class const_iterator
      {
        public:
         const_iterator(T *ptr) : ptr_{ptr} {}
         const_iterator operator++() { auto i = *this; ++ptr_; return i; }
         const_iterator operator++(int) { ++ptr_; return *this; }
         const T &operator*() const { return *ptr_; }
         const T *operator->() const { return ptr_; }
         bool operator==(const const_iterator &rhs) { return ptr_ == rhs.ptr_; }
         bool operator!=(const const_iterator &rhs) { return ptr_ != rhs.ptr_; }
        private:
         T *ptr_;
      };

      Array(size_t size) : size_(size), data_{std::make_unique<T[]>(size)} {}
      size_t size() const { return size_; }
      T &operator[](size_t i) { return data_[i]; };
      const T &operator[](size_t i) const { return data_[i]; }
      iterator begin() { return iterator(data_.get()); }
      iterator end() { return iterator(data_.get() + size_); }
      const_iterator cbegin() const { return const_iterator(data_.get()); }
      const_iterator cend() const { return const_iterator(data_.get() + size_); }

     private:
      size_t size_;
      std::unique_ptr<T[]> data_;
    };



    int main(int argc, char *argv[])
    {
      Array<double> points(2);
      points[0] = 55.66;
      points[1] = 95.27;
      for (auto &e : points) {
        std::cout << e << "\n";
      }
      for (auto it = points.cbegin(); it != points.cend(); ++it) {
        std::cout << *it << "\n";
      }
    }

Iterate an Internal Vector
--------------------------

.. code-block:: cpp

    #include <iostream>
    #include <utility>
    #include <vector>

    template<typename T>
    class Vector {
     public:
      using iterator = typename std::vector<T>::iterator;
      using const_iterator = typename std::vector<T>::const_iterator;

      inline iterator begin() noexcept {return v.begin();}
      inline iterator end() noexcept {return v.end();}
      inline const_iterator cbegin() const noexcept {return v.cbegin();}
      inline const_iterator cend() const noexcept {return v.cend();}

      template<class... Args>
      auto emplace_back(Args&&... args) {
          return v.emplace_back(std::forward<Args>(args)...);
      }
     private:
      std::vector<T> v;
    };


    int main(int argc, char *argv[]) {
      Vector<int> v;
      v.emplace_back(1);
      v.emplace_back(2);
      v.emplace_back(3);

      for (auto &it : v) {
          std::cout << it << std::endl;
      }
      return 0;
    }

Iterate a file
--------------

.. code-block:: cpp

    // $ g++ -std=c++17 -Wall -Werror -g -O3 a.cc
    // $ ./a.out file

    #include <iostream>
    #include <iterator>
    #include <fstream>
    #include <string>

    class line : public std::string {};

    std::istream &operator>>(std::istream &is, line &l)
    {
      std::getline(is, l);
      return is;
    }

    class FileReader
    {
     public:
      using iterator = std::istream_iterator<line>;
      inline iterator begin() noexcept { return begin_; }
      inline iterator end() noexcept { return end_; }

     public:
      FileReader(const std::string path) : f_{path}, begin_{f_} {}
      friend std::istream &operator>>(std::istream &, std::string &);

     private:
      std::ifstream f_;
      iterator begin_;
      iterator end_;
    };

    int main(int argc, char *argv[])
    {
      FileReader reader(argv[1]);
      for (auto &line : reader) {
        std::cout << line << "\n";
      }
    }
